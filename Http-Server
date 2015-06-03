#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <sys/socket.h>
#include <sys/types.h>
#include<string.h>
#include<sys/stat.h>
#include<netdb.h>
#include<signal.h>
#include<fcntl.h>
int BYTES=5000, fd, bytes_read;
char  data_to_send[500];
/* Crée une socket en écoute sur le port donné en paramètre. Si le
 * port vaut 0, un numéro de port est automatiquement attribué. En
 * retour, port contient le numéro de port attribué.
 *
 * Le programme s'arrête s'il y a une erreur.
 */
int creer_socket_pour_ecouter (int *port);

/* Attend une connexion via la socket passée en paramètre, et retourne
 * une nouvelle socket correspondant à la connexion acceptée.
 *
 * Provoque l'arrêt du programme s'il y a une erreur.
 */
int accepter_connexion (int sockfd);

/* Retourne une chaîne de caractère donnant l'adresse IP et le numéro
 * de port pour la socket passée en paramètre. Le paramètre who vaut
 * soit LOCAL, soit DISTANT.
 */
enum {LOCAL, DISTANT};
const char *parametres_connexion (int sockfd, int who);


/* La fonction chargée de gérer une connexion sur la socket passée en
 * paramètre.
 */
void gerer_connexion (int sockfd)
{
    int c;
    while(read(sockfd,&c,1)==1){
        write(0,&c,1);

    }
 }
 
 // confd est le socket de connexion
 void manage(int confd)
 {
    int l=0, findRq=0, filefd;
    int i=0;
    char request[256], fileName[512] = "siteWeb";
    char * code400 = "HTTP/1.1 400 BAD REQUEST\r\nContent-Type: text/html\r\n\r\n";
    char * code404 = "HTTP/1.1 404 Not Found\r\nContent-Type: text/html\r\n\r\n";
    char * code500 = "HTTP/1.1 500 Internal Server Error REQUEST\r\nContent-Type: text/html\r\n\r\n";
    char * code200 = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n";
    
    l = readRequest(confd,request,fileName,512);
    if( l < 0 ){
        SendResponseHtmlFile(confd, code400, "file400.html");
    } else if( fileExist(fileName) < 0){
        SendResponseHtmlFile(confd, code404, "file404.html");
    } else if( fileExist(fileName) == 0 ){
        SendResponseHtmlFile(confd, code200, fileName);
    } else {
        SendResponseHtmlFile(confd, code500, "file500.html");
    }
    
    printf("result %d\n", l);
    printf("file name %s\n", fileName);
    printf("request %s\n", request);
    printf("end request\n");
    close(confd);
 }

int fileExist(char file[512]){
     if (open(file, O_RDONLY)!=-1 )    //FILE FOUND
                {return 1;}
     else
         return 0;//FILE not FOUND
} 
void SendResponseHtmlFile(int conffd,char * code,char *fileName){
    if (fileExist(fileName)==1 )    //FILE FOUND
    {
        send(0, "HTTP/1.0 200 OK\n\n", 50, 0);
        while ( (bytes_read=read(open(fileName, O_RDONLY), data_to_send, BYTES))>0 )
            write (1, data_to_send, bytes_read);
    }else
        write(0, "HTTP/1.1/ 404 Not Found\n", 50);//file Not found 
} 
int readRequest(int confd,char *request, char *fileName, int fileNameSize)
{
    int i, j;
    int c = 0 ;
    char buf[1000];
    int result;
    char file[512];
    
    if( (c = recv(confd, buf, sizeof(buf), 0)) > 0)
    {
        buf[c] = '\0';
    }
    
    for(i=0; i<c ; i++)
    {
        request[j] = buf[i];
        j++;
        if(buf[i] == '\n'){
            request[j] = '\0';
        }
    }
    
    result = parseRequest(request, j+1, file, 512);
    fileName = strcpy (fileName, "siteWeb/");
    fileName = strcat (fileName, file);
    
    for(i=0; i<c ; i++)
    {
        request[j] = buf[i];
        j++;
        if(buf[i] == '\n'){
            request[j] = '\0';
        }
    }
    
    return result;
}

// @TODO sendRequestHtmlFile

/* 
 * Fonction principale
 */
int main (void)
{
    int port = 8080;
    int socket_de_connexion;
    int socket_pour_ecouter = creer_socket_pour_ecouter (&port);
    
    fprintf (stderr, "# serveur en écoute sur le port %d\n", port);

    while(1) {
        /* on ne gere qu'une connexion a la fois */

        /* Le serveur attend une connexion sur socket_pour_ecouter.
         * Quand un client se connecte, la connexion est ouverte et le
         * serveur peut échanger avec lui en utilisant la valeur de
         * retour de accepter_connexion : socket_de_connexion.
         */
        socket_de_connexion = accepter_connexion (socket_pour_ecouter);

        fprintf (stderr, "# connexion acceptée pour %s\n",
		 parametres_connexion(socket_de_connexion, DISTANT));

        manage(socket_de_connexion);

        fprintf (stderr, "# connexion raccrochée\n");
    }
}


#define LISTEN_BACKLOG 10

int creer_socket_pour_ecouter (int *port)
{
    int sock;
    struct sockaddr_in addr;
    socklen_t addrlen = sizeof addr;
    int tr;

    if ((sock = socket (PF_INET, SOCK_STREAM, 0)) == -1) {
        perror ("! socket ()");
        exit (1);
    }
    
    if (setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &tr, sizeof(int)) == -1 ){
        perror("setsockopt() SET_REUSEADDR\n");
    }
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons (*port);
    if (bind (sock, (struct sockaddr *)&addr, sizeof addr) == -1) {
        perror ("! bind ()");
        exit (1);
    }
    if (listen (sock, LISTEN_BACKLOG) == -1) {
        perror ("! listen ()");
        exit (1);
    }
    if (getsockname (sock, (struct sockaddr *)&addr, &addrlen) == -1) {
        perror ("! getsockname ()");
        exit (1);
    }
    if (addrlen != sizeof addr) {
        fprintf (stderr, "! getsockname (): addrlen mismatch: %lu -> %lu\n",
                 (unsigned long)sizeof addr, (unsigned long)addrlen);
    }
    *port = ntohs (addr.sin_port);
    return sock;
}


int accepter_connexion (int sockfd)
{
    int newsock;

    if ((newsock = accept (sockfd, NULL, 0)) == -1) {
        perror ("! accept ()");
        exit (1);
    }
    return newsock;
}


const char *parametres_connexion (int sockfd, int who)
{
    static char buf[20];
    struct sockaddr_in addr;
    socklen_t addrlen = sizeof addr;

    sprintf (buf, "(inconnu)");
    switch (who) {
    case LOCAL:
        if (getsockname (sockfd, (struct sockaddr *)&addr, &addrlen) == -1) {
            perror ("! getsockname ()");
            return buf;
        }
        break;
    case DISTANT:
        if (getpeername (sockfd, (struct sockaddr *)&addr, &addrlen) == -1) {
            perror ("! getpeername ()");
            return buf;
        }
        break;
    default:
        fprintf (stderr, "! parametres_connexion (): bad parameter who: %d\n", who);
        return buf;
    }
    if (addrlen != sizeof addr) {
        fprintf (stderr, "! parametres_connexion (): addrlen mismatch: %lu -> %lu\n",
                 (unsigned long)sizeof addr, (unsigned long)addrlen);
    }
    if (addr.sin_family != AF_INET) {
        fprintf (stderr, "! parametres_connexion (): bad family: %d\n",
                 (int )addr.sin_family);
        return buf;
    }
    sprintf (buf, "%s:%d",
             inet_ntoa (addr.sin_addr), (int )ntohs (addr.sin_port));
    return buf;
}

/***************************************************************************************************************************/
int parseRequest(char* requestFromClient, int requestSize, char* string, int stringSize)
{
	char *charPtr[4], *end;
    requestFromClient[requestSize-1]='\0';
    if( (end=strstr(requestFromClient,"\r\n\r\n"))==NULL)
        return(-1);
	*(end+4)='\0';
	charPtr[0]=requestFromClient;
	charPtr[1]=strchr(requestFromClient,' ');	
	if(charPtr[1]==NULL)
	    return(-1);
	charPtr[2]=strchr(charPtr[1]+1,' ');	
	if(charPtr[2]==NULL)
	    return(-1);
	charPtr[3]=strchr(charPtr[2]+1,'\r');	
	if(charPtr[3]==NULL)
	    return(-1);
    *charPtr[1]='\0';
	*charPtr[2]='\0';
	*charPtr[3]='\0';

	if(strcmp(charPtr[0],"GET")!=0) return(-1);
	if(strcmp(charPtr[2]+1,"HTTP/1.1")!=0) return(-1);
	strncpy(string,charPtr[1]+2,stringSize);if(string[stringSize-1]!='\0'){
		fprintf(stderr,"Erreur parseRequest(): la taille de la chaine string n'est pas suffisante (stringSize=%d)\n",stringSize);
		exit(3);
	}
	
	//DEBUG - Vous pouvez le supprimer si vous le souhaitez.
	if( *(charPtr[1]+2) == '\0')
	    fprintf(stderr,"DEBUG-SERVEUR: le nom de fichier demande est vide -\nDEBUG-SERVEUR: - on associe donc le fichier par defaut index.html\n");
	else
	    fprintf(stderr,"DEBUG-SERVEUR: le nom de fichier demande est %s\n",string);

	if( *(charPtr[1]+2) == '\0') strcpy(string,"index.html");

	return(0);
}

/* Affiche au bon format l'adresse IP passer en argument.
 * Il s'agit d'une structure sockaddr_in6 qui peut heberger en pratique une adresse IPv6 
 * ou bien une adresse IPv4 mappee. 
 * Cette fonction est particulierement utile pour afficher les adresses IPs des clients qui
 * se connecte sur un serveur utilisant une socket AF_INET6 (double stack).
*/

void printClientInfo(struct sockaddr_in6 clientAddr)
{
	/* La macro ci-dessous permet de verifier la version du protocole utilise par le client
	*  si c'est de l'ipv4, l'adresse IPv4 est mappee (mapped IPv4 address) dans une adresse ipv6:
	*  sous la forme ::ffff:<IPv4 address>
	*/

	char buffer[40];

	if(IN6_IS_ADDR_V4MAPPED(&clientAddr.sin6_addr))
	{
		fprintf(stderr,"--DEBUG-- IPv4 mapped address\n");
		//Declaration d'une sockaddr IPv4 dans lequel on va faire la copie
		struct sockaddr_in addr4;
        	memset(&addr4,0,sizeof(addr4));
        	addr4.sin_family=AF_INET;
        	addr4.sin_port=clientAddr.sin6_port;
		//On decale de 12 octets - pour arriver aux 4 derniers octets de l'adresse
		//copie-colle des 32 derniers bits de l'adresse IPv6 dans l'adresse IPv4
		//Attention au cast (char *) ci-dessous, necessaire pour decaler de 12 octets et pas 12 fois la taille de s6_addr
        	memcpy(&(addr4.sin_addr.s_addr), (char *) &(clientAddr.sin6_addr.s6_addr)+12,sizeof(addr4.sin_addr.s_addr));

		//Affichage:
		if((char *) inet_ntop(AF_INET,&(addr4.sin_addr),buffer,sizeof(buffer))==NULL)
			perror("Error on function inet_ntop() in the printClientInfo() function");

		fprintf(stderr,"-- Serveur logs -- connection from %s\n",buffer);

	} else { //IPv6
		//Affichage 
		fprintf(stderr,"--DEBUG-- IPv6 address\n");
		if(inet_ntop(AF_INET6,&(clientAddr.sin6_addr),buffer,sizeof(buffer))<0)
			perror("Error on function inet_ntop() in the printClientInfo() function");

		fprintf(stderr,"-- Serveur logs -- connection from %s\n",buffer);
	}


}//fin de printClientInfo()
