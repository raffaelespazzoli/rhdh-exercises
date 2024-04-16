# Lab Preparation

## Start OCP

Request a ["Red Hat OpenShift Container Platform Cluster"](https://demo.redhat.com/catalog?search=openshift&item=babylon-catalog-prod%2Fopenshift-cnv.ocpmulti-wksp-cnv.prod) instance from [Red Hat Demo Platform](https://demo.redhat.com/).

NOTE: You must `cluster-admin` privileges to install the different operators required for this technical exercise.

## Install cert-manager operator

```sh
oc apply -f ./cert-manager-operator.yaml
```

Once the operator is ready you can continue. This command shows the status of this operator:

```sh
on 🎩 ❯ oc get csv -n cert-manager-operator
NAME                            DISPLAY                                       VERSION   REPLACES                        PHASE
cert-manager-operator.v1.13.0   cert-manager Operator for Red Hat OpenShift   1.13.0    cert-manager-operator.v1.12.1   Succeeded
```

## Install gitlab operator

```sh
oc create namespace gitlab-system
oc apply -f https://gitlab.com/api/v4/projects/18899486/packages/generic/gitlab-operator/0.30.1/gitlab-operator-openshift-0.30.1.yaml
```

## Deploy gitlab

```sh
export basedomain=$(oc get ingresscontroller -n openshift-ingress-operator default -o jsonpath='{.status.domain}')
envsubst < ./gitlab.yaml | oc apply -f -
```

Deploying GitLab takes some time, so check its status as `Running` before continuing with next steps:

```sh
oc get gitlabs gitlab -o jsonpath='{.status.phase}' -n gitlab-system
```

GitLab is now accessible with user `root/<password in "gitlab-gitlab-initial-root-password" secret>`. To get the plain
value of that password:

```sh
oc get secret gitlab-gitlab-initial-root-password -o jsonpath='{.data.password}' | base64 -d
```

Setup GitLab with some initial configuration:

```sh
./configure-gitlab.sh
```

this script will do the following:

```
create two groups

- team-a
- team-b

create two users

- user1/@abc1cde2
- user2/@abc1cde2

ensure user1 belongs to team-a and user2 belongs to team-b

create a repo called `sample-app` under `team-a` add the `catalog-info.yaml` and `users-groups.yaml` at the root of the repo.
```

## Install rhdh operator

```sh
oc apply -f ./rhdh-operator.yaml
```

The operator is installed in the `rhdh-operator` namespace:

```sh
oc get csv -n rhdh-operator
```

## Install rhdh instance

```sh
oc apply -f ./rhdh-instance.yaml
```

The instance is deployed in the `rhdh-gitlab` namespace and available at:

```sh
echo https://$(oc get route backstage-developer-hub -n rhdh-gitlab -o jsonpath='{.spec.host}')
```
