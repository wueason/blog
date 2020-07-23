---
title: 并发遍历目录
date: 2020-07-23 15:12:31
tags: [golang,并发,遍历目录]
---

### Talk is cheap, show you the code

为每一个 walkDir 的调用创建一个新的 goroutine。它使用 sync.WaitGroup 来为当前存活的 walkDir 调用计数，一个 goroutine 在计数器减为 0 的时候关闭 fileSizes 通道。

```
package main
import (
    "flag"
    "fmt"
    "io/ioutil"
    "os"
    "path/filepath"
    "sync"
    "time"
)
var verbose = flag.Bool("v", false, "显示详细进度")
func main() {
    // ...确定根目录...
    flag.Parse()
    // 确定初始目录
    roots := flag.Args()
    if len(roots) == 0 {
        roots = []string{"."}
    }
    // 并行遍历每一个文件树
    fileSizes := make(chan int64)
    var n sync.WaitGroup
    for _, root := range roots {
        n.Add(1)
        go walkDir(root, &n, fileSizes)
    }
    go func() {
        n.Wait()
        close(fileSizes)
    }()
    // 定期打印结果
    var tick <-chan time.Time
    if *verbose {
        tick = time.Tick(500 * time.Millisecond)
    }
    var nfiles, nbytes int64
loop:
    for {
        select {
        case size, ok := <-fileSizes:
            if !ok {
                break loop // fileSizes 关闭
            }
            nfiles++
            nbytes += size
        case <-tick:
            printDiskUsage(nfiles, nbytes)
        }
    }
    printDiskUsage(nfiles, nbytes) // 最终总数
}
func printDiskUsage(nfiles, nbytes int64) {
    fmt.Printf("%d files  %.1f GB\n", nfiles, float64(nbytes)/1e9)
}
func walkDir(dir string, n *sync.WaitGroup, fileSizes chan<- int64) {
    defer n.Done()
    for _, entry := range dirents(dir) {
        if entry.IsDir() {
            n.Add(1)
            subdir := filepath.Join(dir, entry.Name())
            go walkDir(subdir, n, fileSizes)
        } else {
            fileSizes <- entry.Size()
        }
    }
}
// sema是一个用于限制目录并发数的计数信号量
var sema = make(chan struct{}, 20)
// dirents返回directory目录中的条目
func dirents(dir string) []os.FileInfo {
    sema <- struct{}{}        // 获取令牌
    defer func() { <-sema }() // 释放令牌
    entries, err := ioutil.ReadDir(dir)
    if err != nil {
        fmt.Fprintf(os.Stderr, "du: %v\n", err)
        return nil
    }
    return entries
}
```