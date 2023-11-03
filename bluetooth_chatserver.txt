#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <bluetooth/bluetooth.h>
#include <bluetooth/hci.h>
#include <bluetooth/hci_lib.h>
#include <bluetooth/rfcomm.h>
#define MAX_BUFFER_SIZE 1024

int scan_connect();
int soc_create();
int main() {
		int ch;
		while(1) {
			printf("\n<-----1--start scanninng and pair to the required device-------->\n");
			printf("\n<-----2-----connect through the socket to chat server------>\n");
			printf("\n<-----3-------------------------------exit----------------->\n");
			printf("\nenter your choice:\n");
			scanf("%d" , &ch);
			switch(ch){
					case 1:
							scan_connect();
							break;
					case 2:
							soc_create();
							break;
					case 3:
							exit(1);
							printf("exiting from the switch_case");
							break;
					default:
							printf("the entry didnt match the not found");
							break;
					}
		}
}
int scan_connect( ) {
		inquiry_info *device = NULL;
        int max_resp;
        int num_resp;
        int dev_id , socket , len , flag;
        int i = 0;
        char addr[20] = {0};
        char name [248] = {0};
		char device_address[18];

        //used to find the local bluetooth adaptor
        dev_id = hci_get_route(NULL);
        printf("the dev id %d\n" ,dev_id);
        if(dev_id < 0){
        	printf("error in opening dev_id\n");
            exit(1);
		}
        //used to open the local power adaptor
        socket = hci_open_dev(dev_id);
        printf(" the socket id :%d\n", socket);
        if(socket < 0) {
        	printf("error in opening socket\n");
            exit(1);
		}
        len = 8;
        max_resp = 255;
        flag = IREQ_CACHE_FLUSH;
        //creating malloc for max no of responses
        //creates malloc for 255 max
        device = (inquiry_info*) malloc (max_resp * sizeof(inquiry_info));
        num_resp = hci_inquiry (dev_id , len , max_resp , NULL , &device , flag);
        printf("total no of response: %d\n", num_resp);
        if(num_resp < 0) {
        	printf("inquiry is not done\n");
            exit(1);
		}
		for(i = 0; i < num_resp; i++) {
        	ba2str(&(device + i)->bdaddr, addr);
            memset(name , 0 , sizeof(name));
            if(hci_read_remote_name(socket , &(device + i)->bdaddr , sizeof(name) , name , 0) < 0)
            	strcpy(name , "[unknown]");
                printf("%d  %s %s\n", i , addr , name);
         }
		printf("enter  the device address in format xx:xx:xx:");
		scanf("%17s" , device_address);
        int device_index = -1;
        bdaddr_t bdaddr;
        str2ba(device_address, &bdaddr);
		printf("%s ", device_address);

        for (int i = 0; i < num_resp; ++i) {
        	if (bacmp(&device[i].bdaddr, &bdaddr) == 0) {
            	device_index = i;
				printf("index is device found:%d\n",device_index);
            	break;
        	}
    	}	
        if(device_index < 0) {
        	printf("device not found\n");
            close(socket);
		}
        uint16_t handle;
        uint16_t ptype = HCI_DM1 | HCI_DM3 | HCI_DM5 | HCI_DH1 | HCI_DH3 | HCI_DH5;
        uint16_t role = HCI_LM_MASTER | HCI_LM_SECURE;
        if(hci_create_connection(socket , &device[device_index].bdaddr,ptype ,  0 , role , &handle , 100000 ) <0 ) {
	    	printf("error in connecting:\n");
            exit(1);
		}else {
			if(hci_authenticate_link(socket, handle , 100000 ) > 0) {
				printf("paired successfully\n");
			}	
			
		}
        free(device);
        close(socket);
        return 0;
}

int soc_create( ) {
    int client_socket, bytes_read;
    struct sockaddr_rc server_address = {0};
    char buffer[MAX_BUFFER_SIZE] = {0};
	char dev_address[18] = {0};
    // Initialize Bluetooth communication
    client_socket = socket(AF_BLUETOOTH, SOCK_STREAM, BTPROTO_RFCOMM);
    if (client_socket < 0) {
        perror("Socket creation failed");
        exit(1);
    }

    // Set server address
    server_address.rc_family = AF_BLUETOOTH;
//      server_address.rc_bdaddr = *BDADDR_ANY;
	printf("enter the server address to connect:"); 
	scanf("%17s" , dev_address);
    str2ba(dev_address, &server_address.rc_bdaddr);
    server_address.rc_channel = (uint8_t)1;// Use channel 1

    // Connect to the server
    if (connect(client_socket, (struct sockaddr *) &server_address, sizeof(server_address)) < 0) {
        perror("Connection failed");
        close(client_socket);
        exit(1);
    }

    // after connecting we can start msgs
    printf("Connected to the server. Start typing messages:\n");
    while (1) {
        printf("You: ");
        fgets(buffer, MAX_BUFFER_SIZE, stdin);
        send(client_socket, buffer, strlen(buffer), 0);

        bytes_read = recv(client_socket, buffer, MAX_BUFFER_SIZE, 0);
        if (bytes_read > 0) {
            buffer[bytes_read] = '\0';
            printf("Server: %s", buffer);
        }
    }

    // Terminate the connection
    close(client_socket);
    return 0;
}


