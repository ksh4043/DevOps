# kubectl get nodes 결과
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   6m16s   v1.37.0

# Pod를 지운 뒤 새 Pod가 생겨난 get pods -w 화면
NAME                         READY   STATUS    RESTARTS   AGE
guestbook-597fbb4f5f-jgnfl   1/1     Running   0          2m4s

# 내 방명록이 쿠버네티스에서 뜬 브라우저 화면
![guestbook 배포 화면](images/guestbook.png)

# 선언형
어떻게 할지를 명령하는 게 아니라 이런 상태였으면 좋겠다를 적어두는 방식


