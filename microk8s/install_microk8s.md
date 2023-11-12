### Install microk8s cluster

1) Ставим snap
```
su root
apt update
apt install snapd
```
2) Устанавливаем microk8s
```
sudo snap install microk8s --classic --channel=1.28
```
3) Добавляем пользователя в группу и назначем владельцем (Делать не из под рута)
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```
4) Проверяем, что ноды доступны и работоспособны
```
microk8s kubectl get nodes
```

//чтобы использовать просто kubectl, вместо microk8s kubectl используйте алиасы
```
alias kubectl='microk8s kubectl'
```

### Install argocd
1) Включаем днс
```
microk8s enable dns && microk8s stop && microk8s start
```
2) Ставим argocd
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
3) Для доступа к аргосд по адресу будем использовать балансировщик metallb
```
microk8s enable metallb
```
При запросе пула адресов вводим свободные адреса в нашей локальной сети, например:
```
Enter each IP address range delimited by comma (e.g. '10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111'): 10.0.10.120-10.0.10.130
```
4) Редактируем дефолтный адвертисмент
```
kubectl edit L2Advertisement default-advertise-all-pools -n metallb-system
```
В нем добавляем default-addresspoll:
```
spec:
  ipAddressPools:
    - default-addresspool
```
5) Готово, чтобы узнать адрес аргосд посмотрите
```
kubectl get svc -n argocd | grep argocd-server
```
