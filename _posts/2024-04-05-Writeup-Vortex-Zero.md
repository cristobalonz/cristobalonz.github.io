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
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(){
	//create a socket
	int network_socket;
	network_socket = socket(AF_INET, SOCK_STREAM, 0);


	//specify an address for the socket
	struct sockaddr_in server_address;
	server_address.sin_family = AF_INET;
	server_address.sin_port = htons(5842);
	inet_aton("13.50.214.90", &server_address.sin_addr.s_addr);

	int connection_status = connect(network_socket,(struct sockaddr *) &server_address, sizeof(server_address));

	//check for error with the connection
	if(connection_status == -1){
		printf("There was an error making a connection to the remote socket \n\n");
	}

	//receive data from the server
	int i;
	unsigned int data = 0;
	unsigned int server_response[4];
	for(i = 0; i < 4 ; i++){
		recv(network_socket, &server_response[i], 4, 0);
		//printf("The server sent the data %u \n", server_response[i]);
	}

	int k;
	for(k = 0; k < 4; k++){
		printf("The server sent the data %u \n", server_response[k]);
		data = data + server_response[k];
	}

	

	printf("The final sum is %lu \n", data);
	
	//Now we send the data to the server and get the passwd
	char buffer[1024];
	send(network_socket,&data,4,0);
	recv(network_socket,buffer,sizeof(buffer),0);
	printf("Auth:\n %s\n",buffer);


	
	//and then close the socket
	close(network_socket);

	
	
	return 0;
}
```

Lo ejecutamos en linux y obtenemos la flag.