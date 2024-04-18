---
title: Writeup Vortex Zero
date: 2024-04-18 12:11:10 +/-TTTT
categories: [OvertheWire]
tags: [linux, overthewire, shellcode, easy]     # TAG names should always be lowercase

image:
  path: /assets/img/Posts/Vortex0/post_img.JPG
  alt: Writeup aero
---

A continuacion tenemos el siguiente desafio de scripting en C de Overthewire en el cual debemos conectarnos al puerto 5842 en la url (votex.labs.overthewire.org)[votex.labs.overthewire.org] recibir 4 numeros binarios sumarlos y volverlos a mandar para recibir la flag. Lo primero que podemos usar es el siguiente script en python2 que resuelve el desafio:

```python
from pwn import *

HOST = 'votex.labs.overthewire.org'
PORT = 5842

s = remote(HOST,PORT)

d =0
for x in xrange(0,4,1): #xrange: lo mismo que range pero mejor manejo de la memoria
    data = s.recvn(4)
    d+= unpack(data, 32, 'little', False) #Convierte un numero binario en entero

d = pack(d & 0xFFFFFFFF, 32, 'little', True) #Convierte un numero entero en binario es little porque el lab es x86
s.send(d)
response = s.recv(1000)
print "Recieved %s" % response


s.close()
```

Pero la idea es aprender C asi que tomamos esa POC y la implementamos en C de la siguiente forma:

```c
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define PORT "3490" // the port client will be connecting to

#define MAXDATASIZE 100 // max number of bytes we can get at once

// get sockaddr, IPv4 or IPv6:
void *get_in_addr(struct sockaddr *sa)
{
    if (sa->sa_family == AF_INET) {
    
    return &(((struct sockaddr_in*)sa)->sin_addr);
    
    }

    return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(int argc, char *argv[])
{

    int sockfd, numbytes;
    char buf[MAXDATASIZE];
    struct addrinfo hints, *servinfo, *p;
    int rv;
    char s[INET6_ADDRSTRLEN];

    if (argc != 2) {
        fprintf(stderr,"usage: client hostname\n");
        exit(1);
    }

    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;

    if ((rv = getaddrinfo(argv[1], PORT, &hints, &servinfo)) != 0) {
        fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
        return 1;
    }

    // loop through all the results and connect to the first we can
    for(p = servinfo; p != NULL; p = p->ai_next) {
        if ((sockfd = socket(p->ai_family, p->ai_socktype,
                p->ai_protocol)) == -1) {
            perror("client: socket");
            continue;
        }

        if (connect(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
            close(sockfd);
            perror("client: connect");
            continue;
        }

        break;
    }

    if (p == NULL) {
        fprintf(stderr, "client: failed to connect\n");
        return 2;
    }

    inet_ntop(p->ai_family, get_in_addr((struct sockaddr *)p->ai_addr),
        s, sizeof s);
    printf("client: connecting to %s\n", s);

    freeaddrinfo(servinfo); // all done with this structure

    if ((numbytes = recv(sockfd, buf, MAXDATASIZE-1, 0)) == -1) {
        perror("recv");
        exit(1);
    }

    buf[numbytes] = '\0';

    printf("client: received '%s'\n",buf);

    close(sockfd);

    return 0;
}
```

Lo ejecutamos en linux y obtenemos la flag.