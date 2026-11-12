# replicas = 5


# 서비스를 통과한 요청에서 서로 다른 Pod hostname 기록
- 5kbtk
- 9ljkg
- dmf4g
- sr7bw
- zr2xw

# Pod 하나를 강제 삭제
sikgong@KSH:~/DevOps$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
guestbook-76fddf6744-5kbtk   1/1     Running   0          17m
guestbook-76fddf6744-9ljkg   1/1     Running   0          17m
guestbook-76fddf6744-dmf4g   1/1     Running   0          17m
guestbook-76fddf6744-sr7bw   1/1     Running   0          17m
guestbook-76fddf6744-zr2xw   1/1     Running   0          17m
rollout-check                0/1     Unknown   0          6d21h
sikgong@KSH:~/DevOps$ kubectl delete pod guestbook-76fddf6744-zr2xw --force --grace-period=0
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "guestbook-76fddf6744-zr2xw" force deleted from default namespace
sikgong@KSH:~/DevOps$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
guestbook-76fddf6744-24jcg   1/1     Running   0          10s
guestbook-76fddf6744-5kbtk   1/1     Running   0          18m
guestbook-76fddf6744-9ljkg   1/1     Running   0          18m
guestbook-76fddf6744-dmf4g   1/1     Running   0          18m
guestbook-76fddf6744-sr7bw   1/1     Running   0          18

# v1 -> v2 롤링 업데이트 후 kubectl rollout history 결과 기록
sikgong@KSH:~/DevOps$ kubectl rollout history deployment guestbook
deployment.apps/guestbook
REVISION  CHANGE-CAUSE
3         <none>
4         <none>

# 롤백 수행
sikgong@KSH:~/DevOps$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
guestbook-76fddf6744   5         5         5       6d22h
guestbook-8599579749   0         0         0       6d21h

sikgong@KSH:~/DevOps$ kubectl rollout undo deployment/guestbook
Warning: resource deployments/guestbook was previously managed with 'kubectl apply'. Rolling back will not update the kubectl.kubernetes.io/last-applied-configuration annotation, which may cause unexpected behavior on future 'kubectl apply' operations. Consider using 'kubectl apply' with your previous configuration file instead.
deployment.apps/guestbook rolled back
sikgong@KSH:~/DevOps$ kubectl rollout status deployment/guestbook
deployment "guestbook" successfully rolled out
sikgong@KSH:~/DevOps$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
guestbook-76fddf6744   5         5         5       6d22h
guestbook-8599579749   0         0         0       6d21h
