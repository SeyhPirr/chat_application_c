- ```c
  int createTCPIpv4Socket() { return socket(AF_INET, SOCK_STREAM, 0); }
  struct sockaddr_in* createIPv4Address(char *ip, int port) {
    struct sockaddr_in *address = malloc(sizeof(struct sockaddr_in));
    address->sin_family = AF_INET;
    address->sin_port = htons(port);
    if(strlen(ip) ==0)
      address->sin_addr.s_addr = INADDR_ANY;
    else
      inet_pton(AF_INET,ip,&address->sin_addr.s_addr);
    return address;
  }
  //ISTEMCI TARAFI KODLARI
  // bu fonksiyon istemciden isim alir ve istemcinin girecegi girdileri dinler 
  // girilen girdileri verilen isimle concatinate ederek sunucuya gondermekle gorevlidir.
  
  void readConsoleEntriesAndSendToServer(int socketFD) {
    char *name = NULL;
    size_t nameSize= 0;
    printf("please enter your name?\n");
    ssize_t nameCount = getline(&name,&nameSize,stdin);
    name[nameCount-1]=0;
    char *line = NULL;
    size_t lineSize= 0;
    printf("type and we will send(type exit)...\n");
    char buffer[1024];
    while(true)
    {
      ssize_t charCount = getline(&line,&lineSize,stdin);
      line[charCount-1]=0;
      sprintf(buffer,"%s:%s",name,line);
      if(charCount>0)
      {
        if(strcmp(line,"exit")==0)
        break;
        ssize_t amountWasSent = send(socketFD,
        buffer,
        strlen(buffer), 0);
      }
    }
  }
  //listen and print fonksiyonu serverden gelicek girdileri dinlemekle gorevlidir
  //eger gelen datanin toplam buyuklugu 0 degilse konsolde gosterir 
  // eger 0 ise dinlemeyi birakir
  //start listenin and print messages fonksiyonu ise listen and print fonksiyonu icin
  // thread olusturmakla gorevlidir. bu sayede hem girdiler dinlenirken
  //hem de sunucudan gelen veriler konsolda gosterilir.
  
  void startListeningAndPrintMessagesOnNewThread(int socketFD) {
      pthread_t id ;
      pthread_create(&id,NULL,listenAndPrint,socketFD);
    }
    void listenAndPrint(int socketFD) {
      char buffer[1024];
      while (true)
      {
        ssize_t amountReceived = recv(socketFD,buffer,1024,0);
        if(amountReceived>0)
        {
          buffer[amountReceived] = 0;
          printf("Response was %s\n ",buffer);
        }
        if(amountReceived==0)
          break;
      }
      close(socketFD);
  }
  
  // socket olusturduktan ve addres struct i doldurulduktan sonra bind edilir
  // baglanti icin server a istek yollanir
  // eger baglanti basariliysa yeni bir thread yaratilarak sunucudan gelen
  // mesajlar dinlenmeye baslanir.
  // mevcut threadde de inputlar dinlenir.
  int main() {
    int socketFD = createTCPIpv4Socket();
    struct sockaddr_in *address = createIPv4Address("127.0.0.1", 2000);
    int result = connect(socketFD,address,sizeof (*address));
    if(result == 0)
    printf("connection was successful\n");
    startListeningAndPrintMessagesOnNewThread(socketFD);
    readConsoleEntriesAndSendToServer(socketFD);
    close(socketFD);
    return 0;
  }
  
  // SUNUCU TARAFI KODLARI
  // baglantiya sahip istemcilerin bilgilerini tutabilmek icin olusturulmus bir yapidir.
  struct AcceptedSocket
  {
    int acceptedSocketFD;
    struct sockaddr_in address;
    int error;
    bool acceptedSuccessfully;
  };
  struct acceptedSocket acceptedSockets[];
  // accept incoming connection fonksiyonundan yararlanir donen socket verilerini 
  // acceptedSockets arrayinde saklar
  // recieve and print incoming data fonksiyonundan yararlanarak yeni client icin
  //ayri bir thread yaratir ve boylece gelen verileri dinleyerek konsolda gosterir.
  void startAcceptingIncomingConnections(int serverSocketFD) {
    while(true)
    {
      struct AcceptedSocket* clientSocket = acceptIncomingConnection(serverSocketFD);
      acceptedSockets[acceptedSocketsCount++] = *clientSocket;
      receiveAndPrintIncomingDataOnSeparateThread(clientSocket);
    }
  }
  //yeni gelen baglanti istegini kabul eder sonrasinda accepted socket struct yapisini
  //doldurur
  struct AcceptedSocket * acceptIncomingConnection(int serverSocketFD) {
    struct sockaddr_in clientAddress ;
    int clientAddressSize = sizeof (struct sockaddr_in);
    int clientSocketFD = accept(serverSocketFD,&clientAddress,&clientAddressSize);
    struct AcceptedSocket* acceptedSocket = malloc(sizeof (struct AcceptedSocket));
    acceptedSocket->address = clientAddress;
    acceptedSocket->acceptedSocketFD = clientSocketFD;
    acceptedSocket->acceptedSuccessfully = clientSocketFD>0;
    if(!acceptedSocket->acceptedSuccessfully)
      acceptedSocket->error = clientSocketFD;
    return acceptedSocket;
  }
  //bu fonksiyon recieve and print incoming data fonksiyonu icin ayri bir thread olusturur
  //yukarda bahsedildigi gibi kabul edilen her bir client icin ayri bir thread olusturulur
  
  void receiveAndPrintIncomingDataOnSeparateThread(struct AcceptedSocket *pSocket) {
    pthread_t id;
    pthread_create(&id,NULL,receiveAndPrintIncomingData,pSocket->acceptedSocketFD);
  }
  //bu fonksiyon istemcilerden gelen verileri kabul edip konsolda gostermekle gorevlidir.
  //send recieved message to other client fonksiyonundan yararlanarak butun istemcilere 
  //gonderir
  void receiveAndPrintIncomingData(int socketFD) {
    char buffer[1024];
    while (true)
    {
    ssize_t amountReceived = recv(socketFD,buffer,1024,0);
    if(amountReceived>0)
    {
    buffer[amountReceived] = 0;
    printf("%s\n",buffer);
    sendReceivedMessageToTheOtherClients(buffer,socketFD);
    }
    if(amountReceived==0)
    break;
    }
    close(socketFD);
  }
  //Bu fonksiyon kabul edilmis butun istemcilere sunucuya gelmis veriyi yollar
  void sendReceivedMessageToTheOtherClients(char *buffer,int socketFD) {
    for(int i = 0 ; i<acceptedSocketsCount ; i++)
      if(acceptedSockets[i].acceptedSocketFD !=socketFD)
      {
        send(acceptedSockets[i].acceptedSocketFD,buffer, strlen(buffer),0);
      }
  }
  // sunucu tarafinin main fonksiyonu  socket olusturduktan address structini doldurduktan
  // sonra bind eder sonrasinda bind ettigi portta dinleyemeye baslar
  // 10 backlog degeriyle dinlemeye baslar
  // kabul ettigi her istemci icin ayri bir thread olusturarak gelen verileri butun
  // istemcilere yollar process bittiginde shutdown eder.
  int main() {
    int serverSocketFD = createTCPIpv4Socket();
    struct sockaddr_in *serverAddress = createIPv4Address("",2000);
    int result = bind(serverSocketFD,serverAddress, sizeof(*serverAddress));
    if(result == 0)
      printf("socket was bound successfully\n");
    int listenResult = listen(serverSocketFD,10);
    startAcceptingIncomingConnections(serverSocketFD);
    shutdown(serverSocketFD,SHUT_RDWR);
    return 0;
  }
  ```