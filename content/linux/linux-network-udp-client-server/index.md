---
title: "Linux网络编程：UDP客户端(Client)和服务端(Server)实现"
date: 2024-03-20T18:40:00+08:00
slug: linux-socket-udp-client-server
categories:
- Linux
tags:
- Linux
- Network
- Socket
- UDP
draft: false
---

## socket api

```c

#include <sys/socket.h>

int socket(int domain, int type, int protocol);

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
int listen(int sockfd, int backlog);
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *addr, socklen_t *addrlen);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);

ssize_t send(int sockfd, const void *buf, size_t len, int flags);
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, 
              const struct sockaddr *addr, socklen_t addrlen);
ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);

```

<br />

## client

```c

#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define msleep(ms) usleep((ms)*1000)

#define SERVER_HOST "0.0.0.0"
#define SERVER_PORT 5566

static int g_app_quit = 0;

int main() {
    struct sockaddr_in server_addr;

    int sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (sock < 0) {
        printf("udp socket failed with [%d]%s", errno, strerror(errno));
        return -1;
    }

    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(SERVER_HOST);
    server_addr.sin_port = htons(SERVER_PORT);

    for (;;) {
        if (g_app_quit) {
            break;
        }
        char buffer[] = "Hello, UDP Server";
        ssize_t wsize = sendto(sock, buffer, strlen(buffer), 0, 
                              (struct sockaddr *) &server_addr, sizeof(server_addr));
        if (wsize < 0) {
            printf("client send failed with [%d]%s", errno, strerror(errno));
            break;
        }
        sleep(1);
    }

    close(sock);
    return 0;
}

```

<br />

## server

```c

#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define msleep(ms) usleep((ms)*1000)

#define SERVER_HOST "0.0.0.0"
#define SERVER_PORT 5566
#define BUFFER_SIZE 1024

static int g_app_quit = 0;

int main() {
    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    int sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (sock < 0) {
        printf("udp socket failed with [%d]%s", errno, strerror(errno));
        return -1;
    }

    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(SERVER_HOST);
    server_addr.sin_port = htons(SERVER_PORT);

    if (bind(sock, (struct sockaddr *) &server_addr, sizeof(server_addr)) != 0) {
        printf("udp bind failed with [%d]%s", errno, strerror(errno));
        close(sock);
        return -1;
    }

    for (;;) {
        if (g_app_quit) {
            break;
        }
        char buffer[BUFFER_SIZE] = "";
        memset(&client_addr, 0, sizeof(client_addr));
        socklen_t addrlen = sizeof(client_addr);
        ssize_t rsize = recvfrom(sock, buffer, BUFFER_SIZE, 0, 
                                (struct sockaddr *) &client_addr, &addrlen);
        if (rsize < 0) {
            printf("client send failed with [%d]%s", errno, strerror(errno));
            break;
        } else if (rsize > 0) {
            printf("receive data: %s from %s:%d", buffer, inet_ntoa(client_addr.sin_addr),
                     ntohs(client_addr.sin_port));
        }
        sleep(1);
    }

    close(sock);
    return 0;
}

```