# Git w praktyce - GitOps

Ten krótki tutorial pokaże nam podstawy podejścia GitOps.
Skorzystamy w nim z możliwości [ArgoCD][argocd],
menedżera wdrożeń oprogramowania dla platformy [Kubernetes][k8s].

[argocd]: https://argo-cd.readthedocs.io/en/stable/
[k8s]: https://kubernetes.io/


## Konfiguracja dostępu do Kubernetesa

Przed przystąpieniem do dalszych kroków, upewnijmy się, że korzystamy z
odpowiedniego klastra Kubernetesa:

```sh
$ kubectl config current-context
docker-desktop
```

Spodziewamy się kontekstu `docker-desktop` lub `rancher-desktop`.
W innym wypadku sprawdźmy listę dostępnych kontekstów:

```sh
$ kubectl config get-contexts -o name
...
docker-desktop
```

Jeśli `docker-desktop` lub `rancher-desktop` nie są widoczne, to sprawdźmy poprawność instalacji
i konfiguracji Docker Desktop lub Rancher Desktop na naszej maszynie.

Jeśli oczekiwany kontekst jest dostępny, to ustawmy go jako domyślny:

```sh
$ kubectl config use-context docker-desktop
Switched to context "docker-desktop".
$ kubectl config current-context
docker-desktop
```

Sprawdźmy czy klaster Kubernetesa działa i jest dostępny:

```sh
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   38m
```

Ze względu na częste używanie polecenia `kubectl` warto zdefiniować do niego alias:

```sh
alias k=kubectl
```

Oraz [autouzupełnianie argumentów aliasu](https://unix.stackexchange.com/a/224228/160506):

```sh
complete -o default -F __start_kubectl k
```


## Fork i lokalna kopia tutorialu

Aby w pełni skorzystać z tego warsztatu (i przećwiczyć pracę z Gitem)
zróbmy własny fork tego repozytorium. W tym celu otwórzmy w przeglądarce
adres `https://github.com/erszcz/phoenix-hello`, upewnijmy się, że jesteśmy
zalogowani na GitHubie, kliknijmy przycisk _Fork_ i na kolejnym ekranie
zielony przycisk _Create fork_.

Po kilku sekundach zostaniemy przekierowani na niemal identyczną stronę repozytorium,
ale tym razem już pod adresem `https://github.com/<twój-login>/phoenix-hello`.
Kolejną różnicą będzie komentarz pod nazwą repozytorium:

```
forked from erszcz/phoenix-hello
```

Ja w dalszej części warsztatu będę korzystał z własnej zdalnej kopii pod
adresem `https://github.com/erszcz/phoenix-hello`, ale każdy z Was
powinien korzystać ze swojej, tj. `https://github.com/<twój-login>/phoenix-hello`,
ponieważ do repozytorium będziemy chcieli też wprowadzać zmiany.

Teraz możemy już sklonować repozytorium z tutorialem na własną maszynę:

```sh
git clone https://github.com/erszcz/git-in-practice.git
# lub
git clone https://github.com/<twój-login>/git-in-practice.git
cd git-in-practice/gitops
```

Poniżej zakładamy, że wszystkie polecenia wykonywane są właśnie
w powyższym katalogu sklonowanego repozytorium.


## Instalacja ArgoCD

### Instalacja w Kubernetesie

Aby korzystać z podejścia GitOps w naszym klastrze Kubernetesa musimy zainstalować oprogramowanie,
które będzie automatyzowało wdrożenia. Tym oprogramowaniem w naszym wypadku jest ArgoCD.

Zanim przystąpimy do instalacji, uruchommy w osobnym terminalu polecenie,
które pozwoli nam na podgląd zasobów na żywo:

```sh
watch kubectl -n argocd get all
```

Następnie zainstalujmy ArgoCD w naszym klastrze:

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f argocd-install.yaml
```

Terminal z podglądem powinien po chwili pokazać nam szereg prawidłowo zainstalowanych zasobów:

```sh
Every 2.0s: kubectl -n argocd get all                                                        x7.local: Fri Mar  8 16:12:13 2024

NAME                                                    READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                     1/1     Running   0          3m37s
pod/argocd-applicationset-controller-5478c64d7c-h498n   1/1     Running   0          3m38s
pod/argocd-dex-server-6b576d67c9-6xctz                  1/1     Running   0          3m38s
pod/argocd-notifications-controller-5f6c747849-tmc8k    1/1     Running   0          3m38s
pod/argocd-redis-76748db5f4-x8d4z                       1/1     Running   0          3m38s
pod/argocd-repo-server-58c78bd74f-lk7mk                 1/1     Running   0          3m37s
pod/argocd-server-5fd847d6bc-z8jsn                      1/1     Running   0          3m37s

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.106.135.23    <none>        7000/TCP,8080/TCP            3m38s
service/argocd-dex-server                         ClusterIP   10.110.141.243   <none>        5556/TCP,5557/TCP,5558/TCP   3m38s
service/argocd-metrics                            ClusterIP   10.100.198.23    <none>        8082/TCP                     3m38s
service/argocd-notifications-controller-metrics   ClusterIP   10.106.65.121    <none>        9001/TCP                     3m38s
service/argocd-redis                              ClusterIP   10.101.121.116   <none>        6379/TCP                     3m38s
service/argocd-repo-server                        ClusterIP   10.97.78.223     <none>        8081/TCP,8084/TCP            3m38s
service/argocd-server                             ClusterIP   10.104.71.154    <none>        80/TCP,443/TCP               3m38s
service/argocd-server-metrics                     ClusterIP   10.105.24.202    <none>        8083/TCP                     3m38s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           3m38s
deployment.apps/argocd-dex-server                  1/1     1            1           3m38s
deployment.apps/argocd-notifications-controller    1/1     1            1           3m38s
deployment.apps/argocd-redis                       1/1     1            1           3m38s
deployment.apps/argocd-repo-server                 1/1     1            1           3m38s
deployment.apps/argocd-server                      1/1     1            1           3m37s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-5478c64d7c   1         1         1       3m38s
replicaset.apps/argocd-dex-server-6b576d67c9                  1         1         1       3m38s
replicaset.apps/argocd-notifications-controller-5f6c747849    1         1         1       3m38s
replicaset.apps/argocd-redis-76748db5f4                       1         1         1       3m38s
replicaset.apps/argocd-repo-server-58c78bd74f                 1         1         1       3m37s
replicaset.apps/argocd-server-5fd847d6bc                      1         1         1       3m37s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     3m37s
```


### Instalacja aplikacji linii poleceń

Na platformie Mac klienta linii poleceń zainstalujemy przez:

```sh
brew install argocd
```

Prawdopodobnie będziemy też musieli dodać modyfikację zmiennej `PATH` do
`~/.bash_profile`, po czym zrestartować terminal:

```sh
echo 'export PATH=/usr/local/Cellar/argocd/2.10.2/bin/:${PATH}' >> ~/.bash_profile
```

Na platformie Linux najłatwiej będzie pobrać gotową binarkę:

```sh
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

Alternatywną, niezależną od platformy i polecaną przez nas metodą jest instalacja przez
menedżer wersji oprogramowania [`asdf`](https://asdf-vm.com/) - niestety jego konfiguracja
wybiega poza zakres tego warsztatu.
Niemniej jednak instalacja różnych narzędzi przy pomocy `asdf` zwykle sprowadza się do:

```sh
asdf plugin add argocd
asdf list-all argocd
# tu dostajemy listę dostępnych wersji
asdf install argocd 2.10.2
asdf local argocd 2.10.2
# to ostatnie polecenie "ustawia" wersję polecenia dla danego katalogu / projektu,
# co jest bardzo wygodne, jeśli często musimy "skakać" pomiędzy projektami... całkiem jak na studiach ;)
```

[Instalacja klienta linii poleceń ArgoCD](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
zawiera bardziej szczegółowe wskazówki dla innych platform i konfiguracji.


### Dostęp do API aplikacji ArgoCD i konfiguracja

Sieć wewnętrzna klastra Kubernetesa domyślnie jest kompletnie izolowana od
sieci hosta, na którym działa klaster. My jednak chcemy aby nasz konsolowy
klient ArgoCD mógł łączyć się z ArgoCD zainstalowanym w klastrze.
W tym celu musimy ustawić tzw. _port forwarding_, czy też
_przekierowanie portów_,
pomiędzy siecią hosta i aplikacją w klastrze - najlepiej w osobnej konsoli:

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

Znak `&` na końcu linii odda kontrolę powłoce,
co pozwoli nam uruchomić więcej operacji forwardingu,
co z kolei przyda się niedługo przy kolejnych usługach.

Po tej operacji powinniśmy móc otworzyć adres `https://localhost:8080` w
przeglądarce internetowej (może pojawić się ostrzeżenie o nieprawidłowym certyfikacie),
zobaczyć ekran logowania i powitalne hasło:

> Let's get stuff deployed!

W ramach tego warsztatu nie będziemy korzystać z aplikacji webowej ArgoCD,
ale zachęcamy do zapoznania się z nią we własnym zakresie.

Zanim będziemy mogli korzystać z ArgoCD musimy się zalogować.
Pobierzmy wygenerowane przy instalacji ArgoCD tymczasowe hasło:

```sh
$ argocd admin initial-password -n argocd

****************

 This password must be only used for first time login.
 We strongly recommend you update the password using `argocd account update-password`.
```

Zalogujmy się za jego pomocą:

```sh
$ argocd login localhost:8080
WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context 'localhost:8080' updated
```

Zmieńmy hasło - w praktyce na silne hasło dla bezpieczeństwa całej instalacji,
tutaj dla wygody na ustalone z góry `1234qwer`:

```sh
$ argocd account update-password
*** Enter password of currently logged in user (admin): ****************
*** Enter new password for user admin: 1234qwer
*** Confirm new password for user admin: 1234qwer
Password updated
Context 'localhost:8080' updated
```

Dodajmy klaster Kubernetesa do klastrów zarządzanych przez naszą
instalację ArgoCD (uruchomioną w tymże klastrze):

```sh
$ argocd cluster add docker-desktop --in-cluster
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `docker-desktop` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0001] ServiceAccount "argocd-manager" already exists in namespace "kube-system"
INFO[0001] ClusterRole "argocd-manager-role" updated
INFO[0001] ClusterRoleBinding "argocd-manager-role-binding" updated
Cluster 'https://kubernetes.default.svc' added
```

Na koniec tego etapu zainstalujmy jeszcze tzw. Reloader, czyli kontroler,
który odpowiada za restart podów, kiedy zmienia się ich konfiguracja
(domyślnie w Kubernetesie tak się nie dzieje, a nam na tym zachowaniu zależy):

```sh
$ kubectl apply -f reloader.yaml
serviceaccount/reloader-reloader created
clusterrole.rbac.authorization.k8s.io/reloader-reloader-role created
clusterrolebinding.rbac.authorization.k8s.io/reloader-reloader-role-binding created
deployment.apps/reloader-reloader created
```

Postęp tej instalacji możemy śledzić w namespace'ie `default` przez:

```sh
watch kubectl -n default get all
```

A wynik końcowy powinien wyglądać mniej więcej tak:

```
Every 2.0s: kubectl -n default get all                          x7.local: Mon Mar 11 10:39:29 2024

NAME                                     READY   STATUS    RESTARTS   AGE
pod/reloader-reloader-5fdf7f56c5-kbsh2   1/1     Running   0          113s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d19h

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/reloader-reloader   1/1     1            1           113s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/reloader-reloader-5fdf7f56c5   1         1         1       113s
```


## Wdrożenie prostej aplikacji webowej przy użyciu ArgoCD

Przyszła pora na stworzenie naszej pierwszej aplikacji zarządzanej przez
ArgoCD w podejściu GitOps
(pamiętaj o zmianie URLa na `https://github.com/<twój-login>/git-in-practice.git`):

```sh
$ argocd app create phoenix-hello --repo https://github.com/erszcz/git-in-practice.git \
    --path gitops/phoenix-hello --dest-server https://kubernetes.default.svc \
    --dest-namespace phoenix-hello
application 'phoenix-hello' created
```

Zwróćmy uwagę na ostatnią opcję - `dest-namespace` - oznacza, że
nasza aplikacja zainstalowana będzie w dedykowanej dla niej przestrzeni nazw
(to dobra praktyka w świecie Kubernetesa).
Podpowiada nam też jak monitorować postęp:

```sh
watch kubectl -n phoenix-hello get all
```

Tę przestrzeń nazw musimy utworzyć samodzielnie:

```sh
$ kubectl create ns phoenix-hello
namespace/phoenix-hello created
```

Samo polecenie `argocd app create` jednak nie instaluje aplikacji, a jedyne tworzy ją w ArgoCD.
Dokonajmy teraz tzw. "synchronizacji", czyli uzgodnienia stanu faktycznego
w klastrze z opisem aplikacji, który ArgoCD znalazło we wskazanych
repozytorium (`--repo`) i ścieżce (`--path`)
(te same pliki mamy też na dysku, jeśli sklonowaliśmy to repozytorium na początku warsztatu):

```sh
$ argocd app sync phoenix-hello
TIMESTAMP                  GROUP        KIND   NAMESPACE                     NAME       STATUS    HEALTH        HOOK  MESSAGE
2024-03-11T10:59:25+01:00   apps  Deployment  phoenix-hello         phoenix-hello     OutOfSync  Missing
2024-03-11T10:59:25+01:00          ConfigMap  phoenix-hello  phoenix-hello-configmap  OutOfSync  Missing
2024-03-11T10:59:25+01:00            Service  phoenix-hello         phoenix-hello     OutOfSync  Missing
2024-03-11T10:59:25+01:00          ConfigMap  phoenix-hello  phoenix-hello-configmap    Synced  Missing
2024-03-11T10:59:25+01:00            Service  phoenix-hello         phoenix-hello    Synced  Healthy
2024-03-11T10:59:25+01:00          ConfigMap  phoenix-hello  phoenix-hello-configmap    Synced   Missing              configmap/phoenix-hello-configmap created
2024-03-11T10:59:25+01:00            Service  phoenix-hello         phoenix-hello       Synced   Healthy              service/phoenix-hello created
2024-03-11T10:59:25+01:00   apps  Deployment  phoenix-hello         phoenix-hello     OutOfSync  Missing              deployment.apps/phoenix-hello created
2024-03-11T10:59:25+01:00   apps  Deployment  phoenix-hello         phoenix-hello    Synced  Progressing              deployment.apps/phoenix-hello created

Name:               argocd/phoenix-hello
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          phoenix-hello
URL:                https://localhost:8080/applications/phoenix-hello
Repo:               https://github.com/erszcz/git-in-practice.git
Target:
Path:               gitops/phoenix-hello
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (311a012)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      311a012d2db482abbf5a2364c1f850f619224c8e
Phase:              Succeeded
Start:              2024-03-11 10:59:25 +0100 CET
Finished:           2024-03-11 10:59:25 +0100 CET
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE      NAME                     STATUS  HEALTH       HOOK  MESSAGE
       ConfigMap   phoenix-hello  phoenix-hello-configmap  Synced                     configmap/phoenix-hello-configmap created
       Service     phoenix-hello  phoenix-hello            Synced  Healthy            service/phoenix-hello created
apps   Deployment  phoenix-hello  phoenix-hello            Synced  Progressing        deployment.apps/phoenix-hello created
```

Stan docelowy monitorowany przez `watch kubectl ...` powinien wyglądać teraz mniej więcej tak:

```sh
Every 2.0s: kubectl -n phoenix-hello get all                    x7.local: Mon Mar 11 11:00:26 2024

NAME                                 READY   STATUS    RESTARTS   AGE
pod/phoenix-hello-7f56db7994-fz27m   1/1     Running   0          61s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/phoenix-hello   ClusterIP   10.98.212.165   <none>        80/TCP    61s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/phoenix-hello   1/1     1            1           61s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/phoenix-hello-7f56db7994   1         1         1       61s
```

Sprawdźmy czy nasza aplikacja działa - w tym celu musimy przekierować
kolejny port (`service/phoenix-hello` pokazuje nam który):

```sh
kubectl -n phoenix-hello port-forward service/phoenix-hello 8088:80 &
```

Pod lokalnym adresem `http://localhost:8088/` powinniśmy zobaczyć teraz
powitalne hasło Phoenix Framework:

```
Peace of mind from prototype to production.
```

A pod adresem `http://localhost:8088/hello` testowy komunikat:

```
Hello, world!
```

Jeśli wszystko się zgadza, to gratulacje!
To nasza pierwsza aplikacja skonfigurowana i zarządzana przez ArgoCD!


## Wdrożenie zmiany w konfiguracji aplikacji dzięki GitOps

W poprzednim kroku synchronizacji aplikacji dokonaliśmy ręcznie,
jednak istotą podejścia GitOps jest automatyzacja.
Skonfigurujmy ArgoCD tak, by samo nadzorowało czy stan aplikacji w
klastrze Kubernetesa odpowiada jej definicji w repozytorium Git:

```sh
argocd app set phoenix-hello --sync-policy automated
```

Domyślnie interwał automatycznej synchronizacji to 3 minuty.
Dokonajmy zmiany w konfiguracji naszej aplikacji przez edycję pliku
`phoenix-hello/phoenix-hello-configmap.yaml` i podmianę np. swojego
imienia w miejsce `world` (nie musimy zostawiać komentarza):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: phoenix-hello-configmap
data:
  # HELLO_NAME_TO_GREET: "world"
  HELLO_NAME_TO_GREET: "Radek"
```

Po zapisaniu zmiany musimy ją skomitować i spuszować - ArgoCD śledzi zdalne repozytorium,
a nie naszą lokalną kopię roboczą.

Po upływie 3 minut nasza aplikacja powinna już z pewnością być zrestartowana.
Będzie to widać np. w kolumnie `AGE` poda `pod/phoenix-hello-*` w wyniku
polecenia `watch kubectl -n phoenix-hello get all`.
Niestety, restart powoduje też zerwanie przekierowania portów, więc
tę operację musimy powtórzyć:

```sh
kubectl -n phoenix-hello port-forward service/phoenix-hello 8088:80 &
```

Ale po tym pod adresem `http://localhost:8088/hello` komunikat powinien wyświetlać już nasze imię:

```
Hello, Radek!
```

**Gratulacje! Tak działa GitOps w praktyce, czyli automatyzacja wdrożeń sterowana Gitem.**

Oczywiście nasza zmiana to tylko przykład ograniczony do konfiguracji aplikacji (edytowaliśmy
tzw. _config map_, mapowanie konfiguracji).
Podobnie jednak moglibyśmy zmodyfikować inne dostępne opcje, parametry zasobów Kubernetesowych
(np. tzw. _health checks_), a nawet obraz uruchamianego kontenera,
czyli w praktyce uruchomić nową wersję aplikacji.
To z kolei otwiera drogę do wprowadzenia dowolnej zmiany w kodzie i logice aplikacji.
W podejściu pełnej automatyzacji DevOps budowanie obrazu zwykle również
jest zautomatyzowane (np. za pomocą GitHub Actions),
a ten temat wykracza już poza tematyczne i czasowe ramy tego warsztatu.


## Odnośniki

Tutorial ten opiera się na oficjalnej dokumentacji [ArgoCD][argocd].

Środowiskiem wdrożeniowym jest [Kubernetes](https://kubernetes.io/).

Testowy kontener to trywialna aplikacja utworzona w [Eliksirze](https://elixir-lang.org/)
z wykorzystaniem [Phoenix Framework](https://phoenixframework.org/).
