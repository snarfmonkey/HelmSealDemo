A quick intro to Helm, Kubeseal, and Encrypted secrets
======================================================

Prereqs:

1. Install minikube and kubectl, the kubernetes control binary
2. Install the sealed secrets controller onto minikube `kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.15.0/controller.yaml`
3. Install the kubeseal binary.
4. Install helm 3 https://helm.sh/docs/intro/install/

Let's try making a sealed secret:
1. First we make a secret (k8s object, manifest, whatever) file, but we don't put this secret in the cluster.
   It's type is generic, and it's name will be foo-secret. it will contain both our supersecretfiles
   `kubectl --namespace default create secret generic foo-secret --from-file supersecretfile1 --from-file supersecretfile2 --dry-run -oyaml > foo-secret.yaml`
2. Then we'll feed out newly-minted secret manifest into kubeseal, and attain a resulting *sealed* secret
   `kubeseal --format yaml --namespace default < foo-secret.yaml > sealed-foo-secret.yaml`
3. We could apply this secret to the cluster with `kubectl apply -f sealed-foo-secret.yaml` but we want to get fancier!
   `cat sealed-foo-secret.yaml` to get
```
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: foo-secret
  namespace: staging
spec:
  encryptedData:
    supersecrefile_1: AgAXXoEKUWWQTYDfsVgObS/WAsinEtGXghqG0wR/7Le2QO8QwRmp9IL5FJ4CQnuz+ZWo8NpH0weMfkpXAlQLrq6tZLrzMBmQGui+evwjXEnyFQxOVvK1bXzweHhHDeoJHltI0TY0/ZbfkyCHolDlNT/SAF32gIcMlUbwAgsEEthfeEqV6aqE7sNW+e2rTCSG8L3aIOiwlrsrSjCAzTT/tTlKLMsPHHiDTTApSk3bhqA2JLcMyS/5VLft2y2wAw7TeUDh7Zb2pQntbnIKwOCXsJj6AmJgvtNBuZnkMfcgLRF3kPc1bEBSX4fvWTOKAR30yRaaA4cNAVvCq0j6+xK4lni64alquZR+yU83FX1LlFgA61jW9kh+UV+K02dsUP0NAuNMG1Nd0Xd1mS/5K0rmbeO6K3lnkuK0tLP7LY7f/CIJHMD7IYh9Wniih5Ai8phfvCupS/On6Mq2RihZzjbhPepDgBxLeX5uQfvuycwJGvPhTe+Apu0NLaeKejYlR2BFCSDGyaPHRS6gTKqLwCnrJRknQCEu58rnFJ49wlWwEdHWejR3//oxz0bE0/c2s9TZTTYrTXKpaXEnm/2/xQUkol7xX+YWVNWenHzlM257dl5keL/nITNg1YAiYM913/xFmAjJtIlDFLSFFpwunG17VdgbaCXUaeaQq445wJa0VyerwErC6GOhBZMWYSyT98NZHhTw3hxtLNQ6sbeeBnnsPGw2ocerLBG5u/kp74DWM7RVr2cUj/Aivv3ZYucjNw/kKt8flQ==
    supersecretfile_2: AgAXXoEKUWWQTYDfsVgObS/WAsinEtGXghqG0wR/7Le2QO8QwRmp9IL5FJ4CQnuz+ZWo8NpH0weMfkpXAlQLrq6tZLrzMBmQGui+evwjXEnyFQxOVvK1bXzweHhHDeoJHltI0TY0/ZbfkyCHolDlNT/SAF32gIcMlUbwAgsEEthfeEqV6aqE7sNW+e2rTCSG8L3aIOiwlrsrSjCAzTT/tTlKLMsPHHiDTTApSk3bhqA2JLcMyS/5VLft2y2wAw7TeUDh7Zb2pQntbnIKwOCXsJj6AmJgvtNBuZnkMfcgLRF3kPc1bEBSX4fvWTOKAR30yRaaA4cNAVvCq0j6+xK4lni64alquZR+yU83FX1LlFgA61jW9kh+UV+K02dsUP0NAuNMG1Nd0Xd1mS/5K0rmbeO6K3lnkuK0tLP7LY7f/CIJHMD7IYh9Wniih5Ai8phfvCupS/On6Mq2RihZzjbhPepDgBxLeX5uQfvuycwJGvPhTe+Apu0NLaeKejYlR2BFCSDGyaPHRS6gTKqLwCnrJRknQCEu58rnFJ49wlWwEdHWejR3//oxz0bE0/c2s9TZTTYrTXKpaXEnm/2/xQUkol7xX+YWVNWenHzlM257dl5keL/nITNg1YAiYM913/xFmAjJtIlDFLSFFpwunG17VdgbaCXUaeaQq445wJa0VyerwErC6GOhBZMWYSyT98NZHhTw3hxtLNQ6sbeeBnnsPGw2ocerLBG5u/kp74DWM7RVr2cUj/Aivv3ZYucjNw/kKt8flQ==
  template:
    metadata:
      creationTimestamp: null
      name: foo-secret
      namespace: staging
```
4. Copy and paste the supersecretfile_1: and supersecretfile_2: lines into ./values.yaml with a 2 space indent, right under `sealedSecrets`
   so it looks like
```
sealedSecrets:
  supersecrefile_1: AgAXXoEKUWWQTYDfsVgObS/WAsinEtGXghqG0wR/7Le2QO8QwRmp9IL5FJ4CQnuz+ZWo8NpH0weMfkpXAlQLrq6tZLrzMBmQGui+evwjXEnyFQxOVvK1bXzweHhHDeoJHltI0TY0/ZbfkyCHolDlNT/SAF32gIcMlUbwAgsEEthfeEqV6aqE7sNW+e2rTCSG8L3aIOiwlrsrSjCAzTT/tTlKLMsPHHiDTTApSk3bhqA2JLcMyS/5VLft2y2wAw7TeUDh7Zb2pQntbnIKwOCXsJj6AmJgvtNBuZnkMfcgLRF3kPc1bEBSX4fvWTOKAR30yRaaA4cNAVvCq0j6+xK4lni64alquZR+yU83FX1LlFgA61jW9kh+UV+K02dsUP0NAuNMG1Nd0Xd1mS/5K0rmbeO6K3lnkuK0tLP7LY7f/CIJHMD7IYh9Wniih5Ai8phfvCupS/On6Mq2RihZzjbhPepDgBxLeX5uQfvuycwJGvPhTe+Apu0NLaeKejYlR2BFCSDGyaPHRS6gTKqLwCnrJRknQCEu58rnFJ49wlWwEdHWejR3//oxz0bE0/c2s9TZTTYrTXKpaXEnm/2/xQUkol7xX+YWVNWenHzlM257dl5keL/nITNg1YAiYM913/xFmAjJtIlDFLSFFpwunG17VdgbaCXUaeaQq445wJa0VyerwErC6GOhBZMWYSyT98NZHhTw3hxtLNQ6sbeeBnnsPGw2ocerLBG5u/kp74DWM7RVr2cUj/Aivv3ZYucjNw/kKt8flQ==
  supersecretfile_2: AgAXXoEKUWWQTYDfsVgObS/WAsinEtGXghqG0wR/7Le2QO8QwRmp9IL5FJ4CQnuz+ZWo8NpH0weMfkpXAlQLrq6tZLrzMBmQGui+evwjXEnyFQxOVvK1bXzweHhHDeoJHltI0TY0/ZbfkyCHolDlNT/SAF32gIcMlUbwAgsEEthfeEqV6aqE7sNW+e2rTCSG8L3aIOiwlrsrSjCAzTT/tTlKLMsPHHiDTTApSk3bhqA2JLcMyS/5VLft2y2wAw7TeUDh7Zb2pQntbnIKwOCXsJj6AmJgvtNBuZnkMfcgLRF3kPc1bEBSX4fvWTOKAR30yRaaA4cNAVvCq0j6+xK4lni64alquZR+yU83FX1LlFgA61jW9kh+UV+K02dsUP0NAuNMG1Nd0Xd1mS/5K0rmbeO6K3lnkuK0tLP7LY7f/CIJHMD7IYh9Wniih5Ai8phfvCupS/On6Mq2RihZzjbhPepDgBxLeX5uQfvuycwJGvPhTe+Apu0NLaeKejYlR2BFCSDGyaPHRS6gTKqLwCnrJRknQCEu58rnFJ49wlWwEdHWejR3//oxz0bE0/c2s9TZTTYrTXKpaXEnm/2/xQUkol7xX+YWVNWenHzlM257dl5keL/nITNg1YAiYM913/xFmAjJtIlDFLSFFpwunG17VdgbaCXUaeaQq445wJa0VyerwErC6GOhBZMWYSyT98NZHhTw3hxtLNQ6sbeeBnnsPGw2ocerLBG5u/kp74DWM7RVr2cUj/Aivv3ZYucjNw/kKt8flQ==

replicaCount: 1

image:
...
```
5. use these values to create manifests for kubernetes, including the sealed-secret:
   `helm template ./ -f /values.yaml`
6. If you want to get crazy, you can go ahead and install it into minikube:
    `helm install ./ -f /values.yaml --debug`
7. If everything goes to plan, it should put like, an nginx pod in your minikube's default namespace
   `kubectl get po -n default` should reveal the pod names.
8. "Enter" the pod
   `kubectl exec -it kubesealdemo-665bffdc8c-lkcqz -- bash`   Note: Your pod name will be different
9. Observe resulting secrets
   `ls /etc/secrets`
