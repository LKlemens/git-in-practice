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
$ kubectl config get-contexts
CURRENT   NAME              CLUSTER             AUTHINFO            NAMESPACE
...       ...               ...                 ...                 ...
*         docker-desktop    docker-desktop      docker-desktop
```

Jeśli `docker-desktop` lub `rancher-desktop` nie są widoczne,
to sprawdźmy poprawność instalacji i konfiguracji Docker Desktop lub Rancher Desktop.
Jeśli pożądany kontekst jest dostępny, to ustawmy go jako domyślny:

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


## Lokalna kopia tutorialu

Aby przećwiczyć pracę z Gitem, możemy sklonować repozytorium z tutorialem na własną maszynę:

```sh
git clone https://github.com/erszcz/git-in-practice.git
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

Terminal z podglądem powinien pokazać nam szereg prawidłowo zainstalowanych zasobów:

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


## Instalacja aplikacji linii poleceń

Na platformie Mac klienta linii poleceń zainstalujemy przez:

```sh
brew install argocd
```

Na platformie linuksowej najłatwiej będzie pobrać gotową binarkę:

```sh
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

Alternatywną, polecaną przez nas metodą jest menedżer wersji
oprogramowania [`asdf`](https://asdf-vm.com/) - niestety jego konfiguracja
wybiega poza zakres tego warsztatu.
Niemniej jednak instalacja różnych narzędzi przy pomocy `asdf` zwykle sprowadza się do:

```sh
asdf plugin add argocd
asdf list-all argocd
# tu dostajemy listę dostępnych wersji
asdf install argocd 2.10.2
asdf local argocd 2.10.2
```

[Instalacja klienta linii poleceń ArgoCD](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
zawiera bardziej szczegółowe wskazówki dla innych platform i konfiguracji.


## Wdrożenie prostej aplikacji webowej przy użyciu ArgoCD


## Wdrożenie zmiany w konfiguracji aplikacji dzięki GitOps


## Odnośniki

Tutorial ten opiera się na oficjalnej dokumentacji [ArgoCD][argocd].
