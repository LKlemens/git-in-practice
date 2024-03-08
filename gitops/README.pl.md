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


## Instalacja ArgoCD

[Instalacja ArgoCD]() zawiera bardziej szczegółowe wskazówki dla innych platform.


## Wdrożenie prostej aplikacji webowej przy użyciu ArgoCD


## Wdrożenie zmiany w konfiguracji aplikacji dzięki GitOps


## Odnośniki

Tutorial ten opiera się na oficjalnej dokumentacji [ArgoCD][argocd].
