---
title: Securing Datahub on k8s
date: 2022-08-09 18:00
categories: [Data Engineering]
tags: [data engineering,data discovery,data catalog,k8s,kubernetes,notes]
---

I found myself having to deploy [Datahub](https://datahubproject.io/) on
a kubernetes cluster. Deploying the application is quite straight forward using
[Helm](https://helm.sh). I did run into a tiny issue that stopped me in my tracks
for a moment being somewhat new to kubernetes and helm. Datahub comes with a default
account that has full administrator access to the application. The application
provides various mechanisms for authentication (e.g. OIDC), however, this account can
still be accessed by bypassing whatever authentication is set by simply navigating
directly to the `/login` endpoint. Fortunately, this account can be overriden by
[providing a properties file](https://datahubproject.io/) with a different password
from the default password.

Now, in kubernetes we can [mount](https://kubernetes.io/docs/concepts/storage/volumes/)
our custom properties file into a running pod. I had an idea of how to do this but I was
a bit lost on how I could do this with helm involved. My understanding of how to mount
a volume to a pod revolved around including the volume information in the pod definition.
Being that I am working with a helm chart, I figured that the pod definition was
actually within the helm chart so I had no direct access to it. Stupidly, I went ahead
and started going through the helm chart's
[source code](https://github.com/acryldata/datahub-helm) hoping to find something that
may help. That's when I realised that the wonderful datahub guys provided a way for users
of the chart to extend the volumes definition for the pod. Worse this information was
kind of there in the chart's documentation but because I didn't thoroughly go through
the effin-manual I missed it. Anyway, the datahub chart provides two configuration
parameters that can be used for what I needed. So here is how I went about it:

1. Create a user.props file with a new password for the root account. The file looked
something like the following:

    ```properties
    datahub=new-password
    ```

2. Create a config map from the properties file above:

    ```sh
    kubectl create configmap datahub-volumes --from-file=user.props
    ```

3. Add the volume mount to my helm values file using the parameters provided by the chart.
I ended up with a values file that looks something like this:

    ```yaml
    # values.yaml
    datahub-frontend:
      enabled: true
      extraVolumes:
        name: user-props-volume
        configmap:
          name: datahub-volumes
      extraVolumeMounts:
        - name: user-props-volume
          mountPath: /datahub-frontend/conf/user.props
          subPath: user.props
    ```

4. Install datahub

    ```sh
    helm install datahub datahub/datahub --values values.yaml
    ```

A bit of an explanation of what is going on here... Volumes can be created from
a number of sources including CnfigMaps. So, I created a ConfigMap in step (2) that
contains the user.props file that I need to mount. In step (3) I am using the
`datahub-frontend.extraVolumes` and `datahub-frontend.extraVolumeMount` parameters
provided by the datahub chart to add more volumes to one of datahub's pods.
Under extraVolumes I define the volume under name user-props-volume and specify that
this volume is from a configmap named datahub-volumes. And in extraVolumeMounts,
I am telling kubernetes to mount a file named user.props which exists in volume
datahub-volumes at path /datahub-frontend/conf/user.props. The `subPath`
parameter specifies the name of the file I want to mount instead of mounting the
entire volume as a directory.

