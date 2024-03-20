---
title: "Linux网络编程：常用API"
date: 2024-03-20T18:40:00+08:00
slug: linux-socket-api-summary
categories:
- Linux
tags:
- Linux
- Network
- Socket
draft: false
---

## socket basic api

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

int getpeername(int sockfd, struct sockaddr *addr, socklen_t addrlen);
int getsockname(int sockfd, struct sockaddr *addr, socklen_t addrlen);

int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t optlen);
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

```

<br />

## socket address converts

[https://www.man7.org/linux/man-pages/man3/inet.3.html](https://www.man7.org/linux/man-pages/man3/inet.3.html)

```c

#include <arpa/inet.h>

int inet_aton(const char *cp, struct in_addr *inp);
char *inet_ntoa(struct in_addr in);

int inet_pton(int af, const char *src, void *dst);
const char *inet_ntop (int af, const void *cp, char *buf, socklen_t len);

in_addr_t inet_addr(const char *cp);
in_addr_t inet_network(const char *cp);

```

<br />

## convert values between host and network byte order

[https://www.man7.org/linux/man-pages/man3/htonl.3.html](https://www.man7.org/linux/man-pages/man3/htonl.3.html)

```c

#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);

```

<br />

## network address and service translation

[https://www.man7.org/linux/man-pages/man3/getaddrinfo.3.html](https://www.man7.org/linux/man-pages/man3/getaddrinfo.3.html)

```c

#include <netdb.h>

int getnameinfo(const struct sockaddr *addr, socklen_t addrlen,
                char *host, socklen_t hostlen,
                char *serv, socklen_t servlen,
                int flags);

int getaddrinfo(const char *node, const char *service,
                const struct addrinfo *hints,
                struct addrinfo **res);

void freeaddrinfo(struct addrinfo *res);

struct addrinfo {
    int              ai_flags;
    int              ai_family;
    int              ai_socktype;
    int              ai_protocol;
    socklen_t        ai_addrlen;
    struct sockaddr *ai_addr;
    char            *ai_canonname;
    struct addrinfo *ai_next;
};

```

<br />

## socket address

[https://www.man7.org/linux/man-pages/man3/sockaddr.3type.html](https://www.man7.org/linux/man-pages/man3/sockaddr.3type.html)

```c

#include <sys/socket.h>

struct sockaddr {
    sa_family_t     sa_family;      /* Address family */
    char            sa_data[];      /* Socket address */
};

struct sockaddr_storage {
    sa_family_t     ss_family;      /* Address family */
};

typedef /* ... */ socklen_t;
typedef /* ... */ sa_family_t;

/// Internet domain sockets
#include <netinet/in.h>

struct sockaddr_in {
    sa_family_t     sin_family;     /* AF_INET */
    in_port_t       sin_port;       /* Port number */
    struct in_addr  sin_addr;       /* IPv4 address */
};

struct sockaddr_in6 {
    sa_family_t     sin6_family;    /* AF_INET6 */
    in_port_t       sin6_port;      /* Port number */
    uint32_t        sin6_flowinfo;  /* IPv6 flow info */
    struct in6_addr sin6_addr;      /* IPv6 address */
    uint32_t        sin6_scope_id;  /* Set of interfaces for a scope */
};

struct in_addr {
    in_addr_t s_addr;
};

struct in6_addr {
    uint8_t   s6_addr[16];
};

typedef uint32_t in_addr_t;
typedef uint16_t in_port_t;

/// UNIX domain sockets
#include <sys/un.h>

struct sockaddr_un {
    sa_family_t     sun_family;     /* Address family */
    char            sun_path[];     /* Socket pathname */
};

```

<br/>