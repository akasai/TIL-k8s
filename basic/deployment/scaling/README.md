# Scaling

## scale

```bash
$ kubectl scale deployment/<name> --replicas=<n>
```

## autoscale

`horizontalpodautoscaler` 가 생성되며 조건에 맞게 자동으로 스케일링 된다.

강제로 replica를 min보다 낮게 세팅해도 min으로 자동으로 스케일링 된다.

```bash
$ kubectl autoscale deployment/<name> --min=<n> --max=<n> --cpu-percent=<n> 
```
