k create svc nodeport web-service --tcp=8080:8080

to create a daemonset manifest, create a deployment manifest - [replicas && stategy]

can i use this to create manifest for daemonset?? 🤔
  # Start a nginx pod, but overload the spec with a partial set of values parsed from JSON
  kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

to find path of static pod - /var/lib/kubelet/config.yaml > .staticPodPath

no global default for priority class -> new pods get priority class value of 0.

custom scheduler - related to - serviceaccount && clusterrolebinding 📥

find enabled admissions plugins: ps -ef | grep kube-apiserver | grep admission-plugins

 k run webapp-green --image kodekloud/webapp-color --command=false -- --color=green

entrypoint and cmd, the command + the args
