---
title: "Linux网络编程：TCP客户端(Client)和服务端(Server)实现"
date: 2024-03-20T20:20:00+08:00
slug: linux-socket-tcp-client-server
categories:
- Linux
tags:
- Linux
- Network
- Socket
- TCP
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
    int sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(SERVER_HOST);
    server_addr.sin_port = htons(SERVER_PORT);

    if (connect(sock, (struct sockaddr *) &server_addr, sizeof(server_addr)) != 0) {
        printf("connect failed with [%d]%s", errno, strerror(errno));
        close(sock);
        return -1;
    }

    for (;;) {
        if (g_app_quit) {
            break;
        }
        char buffer[] = "Hello, TCP Server";
        ssize_t wsize = send(sock, buffer, strlen(buffer), 0);
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
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <pthread.h>
#include <unistd.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define msleep(ms) usleep((ms)*1000)

#define SERVER_HOST "0.0.0.0"
#define SERVER_PORT 5566
#define BUFFER_SIZE 1024

typedef struct wdm_session_s {
    int sock;
    pthread_t tid;
} wdm_session_t;

static int g_app_quit = 0;

static void *session_process_thread(void *arg) {
    wdm_session_t *session = (wdm_session_t *) arg;
    for (;;) {
        if (g_app_quit) {
            break;
        }
        char buffer[BUFFER_SIZE];
        ssize_t rsize = recv(session->sock, buffer, BUFFER_SIZE, 0);
        if (rsize == -1) {
            printf("session recv failed with [%d]%s", errno, strerror(errno));
            break;
        } else if (rsize == 0) {
            printf("session closed");
            break;
        } else {
            printf("receive data: %s", buffer);
        }
    }
    close(session->sock);
    free(session);
}

int main() {
    struct sockaddr_in server_addr;
    struct sockaddr_in client_addr;

    int server_fd = socket(AF_INET, SOCK_STREAM, 0);

    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(SERVER_HOST);
    server_addr.sin_port = htons(SERVER_PORT);

    if (bind(server_fd, (struct sockaddr *) &server_addr, sizeof(server_addr)) != 0) {
        printf("server bind failed with [%d]%s", errno, strerror(errno));
        close(server_fd);
        return -1;
    }

    if (listen(server_fd, 8) != 0) {
        printf("server listen failed with [%d]%s", errno, strerror(errno));
        close(server_fd);
        return -1;
    }
    printf("server listen on %s:%d ...", SERVER_HOST, SERVER_PORT);

    for (;;) {
        if (g_app_quit) {
            break;
        }

        socklen_t addr_len = sizeof(client_addr);
        int client_fd = accept(server_fd, (struct sockaddr *) &client_addr, &addr_len);
        if (0 < client_fd) {
            printf("client connected from %s:%d",
                   inet_ntoa(client_addr.sin_addr),
                   ntohs(client_addr.sin_port));
            wdm_session_t *session = (wdm_session_t *) malloc(sizeof(wdm_session_t));
            session->sock = client_fd;
            if (pthread_create(&session->tid, NULL, session_process_thread, session) != 0) {
                close(session->sock);
                free(session);
                printf("create session process thread failed with [%d]%s", errno, strerror(errno));
            }
        } else {
            printf("server accept failed with [%d]%s", errno, strerror(errno));
        }

        msleep(30);
    }

    close(server_fd);
    return 0;
}

```