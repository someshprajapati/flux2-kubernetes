# Flux2 on Kubernetes

### 1. Kubernetes
Get a Kubernetes Cluster. In this, I use Docker Desktop for MAC.
### 2. Installing Flux
```
S😎MESH[flux2-kubernetes (master)]~$ brew install fluxcd/tap/flux

S😎MESH[flux2-kubernetes (master)]~$ flux --version
flux version 2.0.0-rc.1

S😎MESH[flux2-kubernetes (master)]~$ flux check --pre
► checking prerequisites
✔ Kubernetes 1.25.9 >=1.20.6-0
✔ prerequisites checks passed
```

### 3. Setting up the Flux
Make sure you are pointing to the kubernetes cluster you want to use.
```
S😎MESH[flux2-kubernetes (master)]~$ kubectx
Switched to context "docker-desktop".

S😎MESH[flux2-kubernetes (master)]~$ kubectl get nodes
NAME             STATUS   ROLES           AGE    VERSION
docker-desktop   Ready    control-plane   5m3s   v1.25.9

S😎MESH[flux2-kubernetes (master)]~$ kubectl get ns
NAME              STATUS   AGE
default           Active   5m34s
kube-node-lease   Active   5m36s
kube-public       Active   5m36s
kube-system       Active   5m36s

export GITHUB_TOKEN=ghp_xyz
export GITHUB_ORG=someshprajapati
export GITHUB_PERSONAL=true

flux bootstrap github \
    --owner $GITHUB_ORG \
    --repository flux2-kubernetes \
    --branch master \
    --path apps \
    --personal $GITHUB_PERSONAL

S😎MESH[flux2-kubernetes (master)]~$ export GITHUB_TOKEN=ghp_xyz
S😎MESH[flux2-kubernetes (master)]~$ export GITHUB_ORG=someshprajapati
S😎MESH[flux2-kubernetes (master)]~$ export GITHUB_PERSONAL=true

S😎MESH[flux2-kubernetes (master)]~$ flux bootstrap github \
>     --owner $GITHUB_ORG \
>     --repository flux2-kubernetes \
>     --branch master \
>     --path apps \
>     --personal $GITHUB_PERSONAL
► connecting to github.com
► cloning branch "master" from Git repository "https://github.com/someshprajapati/flux2-kubernetes.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed sync manifests to "master" ("a67c778a5caf6c8295e70c43dab387ef00ed2386")
► pushing component manifests to "https://github.com/someshprajapati/flux2-kubernetes.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBKTFXEZiCt0Wty64t5P+m9yfM1dcehoHCWYEgFzDm7F+TEPjZ3m1oCXVli5IyPqI2qvLPsaAOyYL6xM6U1h9SBN3yaMmY7okEc5PrasR2rFeKi6POO5pbPr/3RwgIO09Ew==
✔ configured deploy key "flux-system-master-flux-system-./apps" for "https://github.com/someshprajapati/flux2-kubernetes"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "master" ("b362c3686a73970096bacfe93660e2d9968db650")
► pushing sync manifests to "https://github.com/someshprajapati/flux2-kubernetes.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

After this step the below files got created in repo:
https://github.com/someshprajapati/flux2-kubernetes/tree/master/apps/flux-system

Pull the flux system related files from remote repo which created by flux bootstrap.
```
S😎MESH[flux2-kubernetes (master)]~$ git pull
 create mode 100644 apps/flux-system/gotk-components.yaml
 create mode 100644 apps/flux-system/gotk-sync.yaml
 create mode 100644 apps/flux-system/kustomization.yaml

S😎MESH[flux2-kubernetes (master)]~$ ls -l apps/flux-system/
total 752
-rw-r--r--@ 1 someshp  staff   366K May  3 21:31 gotk-components.yaml
-rw-r--r--@ 1 someshp  staff   550B May  3 21:31 gotk-sync.yaml
-rw-r--r--@ 1 someshp  staff   115B May  3 21:31 kustomization.yaml


S😎MESH[flux2-kubernetes (master)]~$ kubectl get ns
NAME              STATUS   AGE
default           Active   11m
flux-system       Active   3m54s
kube-node-lease   Active   11m
kube-public       Active   11m
kube-system       Active   11m

S😎MESH[flux2-kubernetes (master)]~$ kgp -n flux-system
NAME                                       READY   STATUS    RESTARTS   AGE
helm-controller-746889d448-h67dr           1/1     Running   0          4m12s
kustomize-controller-99c4d84bd-gfm7q       1/1     Running   0          4m12s
notification-controller-66d964799d-hvq9w   1/1     Running   0          4m12s
source-controller-7ccc5d646-j44wq          1/1     Running   0          4m12s
```

### 4. Creating Staging sources
```
flux create source git staging \
    --url https://github.com/$GITHUB_ORG/flux-staging \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/staging.yaml

flux create kustomization staging \
    --source staging \
    --path "./" \
    --prune true \
    --validation client \
    --interval 10m \
    --export \
    | tee -a apps/staging.yaml
```

> Results:
```
S😎MESH[flux2-kubernetes (master)]~$ flux create source git staging \
>     --url https://github.com/$GITHUB_ORG/flux-staging \
>     --branch master \
>     --interval 30s \
>     --export \
>     | tee apps/staging.yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: staging
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: master
  url: https://github.com/someshprajapati/flux-staging

S😎MESH[flux2-kubernetes (master)]~$ git status
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	apps/staging.yaml



S😎MESH[flux2-kubernetes (master)]~$ flux create kustomization staging \
>     --source staging \
>     --path "./" \
>     --prune true \
>     --validation client \
>     --interval 10m \
>     --export \
>     | tee -a apps/staging.yaml
Flag --validation has been deprecated, this arg is no longer used, all resources are validated using server-side apply dry-run
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: staging
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./
  prune: true
  sourceRef:
    kind: GitRepository
    name: staging
```



### 5. Creating Production sources
```
flux create source git production \
    --url https://github.com/$GITHUB_ORG/flux-production \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/production.yaml

flux create kustomization production \
    --source production \
    --path "./" \
    --prune true \
    --validation client \
    --interval 10m \
    --export \
    | tee -a apps/production.yaml
```

> Results:
```
S😎MESH[flux2-kubernetes (master)]~$ flux create source git production \
>     --url https://github.com/$GITHUB_ORG/flux-production \
>     --branch master \
>     --interval 30s \
>     --export \
>     | tee apps/production.yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: production
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: master
  url: https://github.com/someshprajapati/flux-production


S😎MESH[flux2-kubernetes (master)]~$ flux create kustomization production \
>     --source production \
>     --path "./" \
>     --prune true \
>     --validation client \
>     --interval 10m \
>     --export \
>     | tee -a apps/production.yaml
Flag --validation has been deprecated, this arg is no longer used, all resources are validated using server-side apply dry-run
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: production
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./
  prune: true
  sourceRef:
    kind: GitRepository
    name: production

```

### 6. Creating devops-toolkit sources
```
flux create source git \
    devops-toolkit \
    --url https://github.com/vfarcic/devops-toolkit \
    --branch master \
    --interval 30s \
    --export \
    | tee apps/devops-toolkit.yaml
```

> Results:
```
S😎MESH[flux2-kubernetes (master)]~$ flux create source git \
>     devops-toolkit \
>     --url https://github.com/vfarcic/devops-toolkit \
>     --branch master \
>     --interval 30s \
>     --export \
>     | tee apps/devops-toolkit.yaml
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: devops-toolkit
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: master
  url: https://github.com/vfarcic/devops-toolkit
```


### 7. Add the newly created files in steps [4-6] to github repo
```
S😎MESH[flux2-kubernetes (master)]~$ git status

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	apps/devops-toolkit.yaml
	apps/production.yaml
	apps/staging.yaml

S😎MESH[flux2-kubernetes (master)]~$ git add apps/devops-toolkit.yaml apps/production.yaml apps/staging.yaml

S😎MESH[flux2-kubernetes (master)]~$ git commit -m "Added the initial environments file"
[master e2278e0] Added the initial environments file
 3 files changed, 61 insertions(+)
 create mode 100644 apps/devops-toolkit.yaml
 create mode 100644 apps/production.yaml
 create mode 100644 apps/staging.yaml

S😎MESH[flux2-kubernetes (master)]~$ git push
```

### 8. Once push the code to remote repo, run the flux.
```
S😎MESH[flux2-kubernetes (master)]~$ watch flux get sources git

Every 2.0s: flux get sources git                                                                                                             someshp-mbp: Wed May  3 21:46:46 2023

NAME            REVISION                SUSPENDED       READY   MESSAGE
flux-system     master@sha1:e2278e09    False           True    stored artifact for revision 'master@sha1:e2278e09'
```

### 9. Create empty github remote repo and clone locally
```
S😎MESH[Personal_github]~$ git clone git@github.com-personal:someshprajapati/flux-staging.git
S😎MESH[Personal_github]~$ mkdir -p flux-staging/apps

S😎MESH[Personal_github]~$ git clone git@github.com-personal:someshprajapati/flux-production.git
S😎MESH[Personal_github]~$ mkdir -p flux-production/apps
```


### 10. Create new namespaces `production` and `staging`
```
S😎MESH[Personal_github]~$ kubectl create namespace production
namespace/production created

S😎MESH[Personal_github]~$ kubectl create namespace staging
namespace/staging created

S😎MESH[Personal_github]~$ k get ns
NAME              STATUS   AGE
default           Active   40m
flux-system       Active   33m
kube-node-lease   Active   40m
kube-public       Active   40m
kube-system       Active   40m
production        Active   82s
staging           Active   77s
```

### 11. Now we can see the flux is in sync for remote repo `flux-staging` and `flux-production`
```
S😎MESH[Personal_github]~$ watch flux get sources git
Every 2.0s: flux get sources git                                                                                                             someshp-mbp: Wed May  3 22:01:12 2023

NAME            REVISION                SUSPENDED       READY   MESSAGE
devops-toolkit  master@sha1:96218231    False           True    stored artifact for revision 'master@sha1:96218231'
flux-system     master@sha1:cd4f22ed    False           True    stored artifact for revision 'master@sha1:cd4f22ed'
production      master@sha1:0a2f768a    False           True    stored artifact for revision 'master@sha1:0a2f768a'
staging         master@sha1:569d9b99    False           True    stored artifact for revision 'master@sha1:569d9b99'


S😎MESH[Personal_github]~$ watch flux get kustomizations
Every 2.0s: flux get kustomizations                                                                                                          someshp-mbp: Wed May  3 22:02:17 2023

NAME            REVISION                SUSPENDED       READY   MESSAGE
flux-system     master@sha1:cd4f22ed    False           True    Applied revision: master@sha1:cd4f22ed
production      master@sha1:0a2f768a    False           True    Applied revision: master@sha1:0a2f768a
staging         master@sha1:569d9b99    False           True    Applied revision: master@sha1:569d9b99
```

### 12. Deploying the first release in staging
```
S😎MESH[Personal_github]~$ cd ..
S😎MESH[Personal_github]~$ cd flux-staging

echo "image:
    tag: 2.9.9
ingress:
    host: staging.devops-toolkit.$INGRESS_HOST.nip.io" \
    | tee values.yaml

flux create helmrelease \
    devops-toolkit-staging \
    --source GitRepository/devops-toolkit \
    --values values.yaml \
    --chart "helm" \
    --target-namespace staging \
    --interval 30s \
    --export \
    | tee apps/devops-toolkit.yaml
```

> Results:
```
S😎MESH[flux-staging (master)]~$ export INGRESS_HOST=192.168.20.22

S😎MESH[flux-staging (master)]~$ echo "image:
>     tag: 2.9.9
> ingress:
>     host: staging.devops-toolkit.$INGRESS_HOST.nip.io" \
>     | tee values.yaml
image:
    tag: 2.9.9
ingress:
    host: staging.devops-toolkit.192.168.20.22.nip.io


S😎MESH[flux-staging (master)]~$ flux create helmrelease \
>     devops-toolkit-staging \
>     --source GitRepository/devops-toolkit \
>     --values values.yaml \
>     --chart "helm" \
>     --target-namespace staging \
>     --interval 30s \
>     --export \
>     | tee apps/devops-toolkit.yaml
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: devops-toolkit-staging
  namespace: flux-system
spec:
  chart:
    spec:
      chart: helm
      reconcileStrategy: ChartVersion
      sourceRef:
        kind: GitRepository
        name: devops-toolkit
  interval: 30s
  targetNamespace: staging
  values:
    image:
      tag: 2.9.9
    ingress:
      host: staging.devops-toolkit.192.168.20.22.nip.io


S😎MESH[flux-staging (master)]~$ git status
Untracked files:
  (use "git add <file>..." to include in what will be committed)
    apps/
	values.yaml

S😎MESH[flux-staging (master)]~$ rm values.yaml

S😎MESH[flux-staging (master)]~$ git status
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   apps/devops-toolkit.yaml

S😎MESH[flux-staging (master)]~$ git commit -m "Initial commit"
[master f806b35] Initial commit
 1 file changed, 21 insertions(+)
 create mode 100644 apps/devops-toolkit.yaml

S😎MESH[flux-staging (master)]~$ git push
```

### 13. Verifying the first release in staging
```
S😎MESH[flux-staging (master)]~$ watch flux get sources git

S😎MESH[flux-staging (master)]~$ watch flux get helmreleases
Every 2.0s: flux get helmreleases                                                                                                            someshp-mbp: Wed May  3 22:23:50 2023
NAME                    REVISION        SUSPENDED       READY   MESSAGE
devops-toolkit-staging                  False           Unknown Reconciliation in progress


S😎MESH[flux-staging (master)]~$ watch flux get helmreleases
Every 2.0s: flux get helmreleases                                                                                                            someshp-mbp: Wed May  3 22:25:33 2023
NAME                    REVISION        SUSPENDED       READY   MESSAGE
devops-toolkit-staging  0.1.0           False           True    Release reconciliation succeeded


S😎MESH[flux-staging (master)]~$ kubectl get pods -n staging
NAME                                                             READY   STATUS    RESTARTS   AGE
staging-devops-toolkit-staging-devops-toolkit-5bccb778b8-jlrzx   1/1     Running   0          2m49s

S😎MESH[flux-staging (master)]~$ kubectl describe pod staging-devops-toolkit-staging-devops-toolkit-5bccb778b8-jlrzx -n staging | grep Image
    Image:          vfarcic/devops-toolkit-series:2.9.9
    Image ID:       docker-pullable://vfarcic/devops-toolkit-series@sha256:6324c8f0da5dbb0ef0a6f1a0d9c37a2a60070ef37e1344ae1fddd83e88805a4e
```

### 14. Verifying the second release in staging after new image update
```

S😎MESH[flux-staging (master)]~$ vi apps/devops-toolkit.yaml

S😎MESH[flux-staging (master)]~$ git diff
diff --git a/apps/devops-toolkit.yaml b/apps/devops-toolkit.yaml
index 6eae1de..e3cce8f 100644
--- a/apps/devops-toolkit.yaml
+++ b/apps/devops-toolkit.yaml
@@ -16,6 +16,6 @@ spec:
   targetNamespace: staging
   values:
     image:
-      tag: 2.9.9
+      tag: 2.9.17
     ingress:
       host: staging.devops-toolkit.192.168.20.22.nip.io


S😎MESH[flux-staging (master)]~$ git commit -am "Updated the tags"

S😎MESH[flux-staging (master)]~$ git push

S😎MESH[flux-staging (master)]~$ k get pods -n staging
NAME                                                             READY   STATUS              RESTARTS   AGE
staging-devops-toolkit-staging-devops-toolkit-5bccb778b8-jlrzx   1/1     Running             0          6m34s
staging-devops-toolkit-staging-devops-toolkit-76b6cb8665-dz2dn   0/1     ContainerCreating   0          2s


S😎MESH[flux-staging (master)]~$ k get pods -n staging
NAME                                                             READY   STATUS    RESTARTS   AGE
staging-devops-toolkit-staging-devops-toolkit-76b6cb8665-dz2dn   1/1     Running   0          50s

S😎MESH[flux-staging (master)]~$ k describe pod staging-devops-toolkit-staging-devops-toolkit-76b6cb8665-dz2dn -n staging |grep Image
    Image:          vfarcic/devops-toolkit-series:2.9.17
    Image ID:       docker-pullable://vfarcic/devops-toolkit-series@sha256:0b0ce138b002455f0620228437f220e904244c12ce088459f1703c17fa6ffa6b
```

### 15. Deploying the first release in production
```
S😎MESH[Personal_github]~$ cd ..

S😎MESH[Personal_github]~$ cd flux-production/

echo "image:
    tag: 2.9.17
ingress:
    host: devops-toolkit.$INGRESS_HOST.nip.io" \
    | tee values.yaml

flux create helmrelease \
    devops-toolkit-production \
    --source GitRepository/devops-toolkit \
    --values values.yaml \
    --chart "helm" \
    --target-namespace production \
    --interval 30s \
    --export \
    | tee apps/devops-toolkit.yaml
```

> Results:
```
S😎MESH[flux-production (master)]~$ export INGRESS_HOST=192.168.20.22

S😎MESH[flux-production (master)]~$ echo "image:
>     tag: 2.9.17
> ingress:
>     host: devops-toolkit.$INGRESS_HOST.nip.io" \
>     | tee values.yaml
image:
    tag: 2.9.17
ingress:
    host: devops-toolkit.192.168.20.22.nip.io


S😎MESH[flux-production (master)]~$ flux create helmrelease \
>     devops-toolkit-production \
>     --source GitRepository/devops-toolkit \
>     --values values.yaml \
>     --chart "helm" \
>     --target-namespace production \
>     --interval 30s \
>     --export \
>     | tee apps/devops-toolkit.yaml
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: devops-toolkit-production
  namespace: flux-system
spec:
  chart:
    spec:
      chart: helm
      reconcileStrategy: ChartVersion
      sourceRef:
        kind: GitRepository
        name: devops-toolkit
  interval: 30s
  targetNamespace: production
  values:
    image:
      tag: 2.9.17
    ingress:
      host: devops-toolkit.192.168.20.22.nip.io


S😎MESH[flux-production (master)]~$ git status
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	apps/
	values.yaml


S😎MESH[flux-production (master)]~$ rm values.yaml

S😎MESH[flux-production (master)]~$ git add .

S😎MESH[flux-production (master)]~$ git commit -m "Initial commit"

S😎MESH[flux-production (master)]~$ git push
```


### 16. Verifying the first release in staging
```
S😎MESH[flux-production (master)]~$ watch flux get helmreleases

Every 2.0s: flux get helmreleases                                                                                                            someshp-mbp: Thu May  4 20:05:26 2023
NAME                            REVISION        SUSPENDED       READY   MESSAGE
devops-toolkit-production       0.1.0           False           True    Release reconciliation succeeded
devops-toolkit-staging          0.1.0           False           True    Release reconciliation succeeded


S😎MESH[flux-production (master)]~$ k get pods -n production
NAME                                                              READY   STATUS    RESTARTS   AGE
production-devops-toolkit-production-devops-toolkit-64c9d8b7vws   1/1     Running   0          2m4s
```
