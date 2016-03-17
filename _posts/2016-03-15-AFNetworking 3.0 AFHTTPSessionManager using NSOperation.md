---
layout: post
title:  "AFNetworking 3.0 AFHTTPSessionManager using NSOperation"
author: Tony Yang
categories:   AFNetworking
---

* 方式一
{% highlight objc %}
//  ViewController.m

#import "ViewController.h"
#import "AFNetworking.h"
#import "AFHTTPSessionOperation.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    NSString *urlString1 = @"...";
    NSString *urlString2 = @"...";

    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    queue.name = @"AFHTTPSessionManager queue";

    NSOperation *completionOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"All done");
    }];

    NSOperation *op1 = [AFHTTPSessionOperation operationWithManager:manager HTTPMethod:@"GET" URLString:urlString1 parameters:nil uploadProgress:nil downloadProgress:nil success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"finished 1");
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failed 1 - error = %@", error.localizedDescription);
    }];
    [completionOperation addDependency:op1];

    NSOperation *op2 = [AFHTTPSessionOperation operationWithManager:manager HTTPMethod:@"GET" URLString:urlString2 parameters:nil uploadProgress:nil downloadProgress:nil success:^(NSURLSessionDataTask *task, id responseObject) {
        NSLog(@"finished 2");
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"failed 2 - error = %@", error.localizedDescription);
    }];
    [completionOperation addDependency:op2];

    [queue addOperations:@[op1, op2] waitUntilFinished:false];
    [[NSOperationQueue mainQueue] addOperation:completionOperation];  // do this on whatever queue you want, but often you're updating UI or model objects, in which case you'd use the main queue
}

@end
{% endhighlight %}


* 方法二
{% highlight objc %}
//  ViewController.m

#import "ViewController.h"
#import "AFNetworking.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    NSString *urlString1 = @"...";
    NSString *urlString2 = @"...";

    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    dispatch_group_t group = dispatch_group_create();

    dispatch_group_enter(group);
    [manager GET:urlString1 parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"finished 1");
        dispatch_group_leave(group);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"failed 1 - error = %@", error.localizedDescription);
        dispatch_group_leave(group);
    }];

    dispatch_group_enter(group);
    [manager GET:urlString2 parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"finished 2");
        dispatch_group_leave(group);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"failed 2 - error = %@", error.localizedDescription);
        dispatch_group_leave(group);
    }];

    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"All done");
    });
}

@end
{% endhighlight %}
