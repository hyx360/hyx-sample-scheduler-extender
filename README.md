# hyx-sample-scheduler-extender

## extender工作逻辑

- 在podFitsOnNode函数中生成一个100以内的随机数，检查该随机数是否为奇数，如为奇数则批准该节点，否则拒绝批准该节点。

```go
func LuckyPredicate(pod *v1.Pod, node v1.Node) (bool, []string, error) {
    lucky := (rand.Intn(100) % 2) == 1
    if lucky {
        log.Printf("pod %v/%v is lucky to fit on node %v\n", pod.Name, pod.Namespace, node.Name)
        return true, nil, nil
    }
    log.Printf("pod %v/%v is unlucky to fit on node %v\n", pod.Name, pod.Namespace, node.Name)
    return false, []string{LuckyPredFailMsg}, nil
}
```

- 使用prioritize函数为每个节点随机打分，将当前时间戳模10的值与最⼤优先级相加，在两数之和范围内随机取⼀个值，为当前节点赋分。

```go
func prioritize(args schedulerapi.ExtenderArgs)*schedulerapi. HostPriorityList {
    pod := args.Pod
    nodes := args.Nodes.Items

    hostPriorityList := make(schedulerapi.HostPriorityList,len(nodes))
    for i, node := range nodes {
        t_string := strconv.FormatInt(time.Now().Unix()%10, 10)
        t_int, _ := strconv.Atoi(t_string)
        score := rand.Intn(t_int + schedulerapi.MaxPriority)
        log.Printf(luckyPrioMsg, pod.Name, pod.Namespace, score)
        
        hostPriorityList[i] = schedulerapi.HostPriority{
            Host: node.Name,
            Score: score,
        }
    }
    return &hostPriorityList
}
```
