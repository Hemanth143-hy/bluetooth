#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <bluetooth/bluetooth.h>
#include <bluetooth/rfcomm.h>

#define MAX_BUFFER_SIZE 1024
#define BACKLOG 1  // Number of pending connections in the server queue

void handleClient(int client_socket) {
    char buffer[MAX_BUFFER_SIZE] = {0};
	char buffer1[MAX_BUFFER_SIZE] = {0};
    int bytes_read;

    while (1) {
        bytes_read = recv(client_socket, buffer, MAX_BUFFER_SIZE, 0);
        if (bytes_read <= 0) {
            // Client disconnected or error occurred
            break;
        }
        printf("Client: %s", buffer);

		printf("server:");
		fgets(buffer1, MAX_BUFFER_SIZE , stdin);
	

        // Echo the received message back to the client
        send(client_socket, buffer1, bytes_read, 0);

        // Clear the buffer
        memset(buffer1, 0, MAX_BUFFER_SIZE);
		memset(buffer, 0, MAX_BUFFER_SIZE);
    }

    close(client_socket);
}

int main() {
    int server_socket, client_socket;
    struct sockaddr_rc server_address = {0};
    struct sockaddr_rc client_address = {0};
    socklen_t address_length;
    char buffer[MAX_BUFFER_SIZE] = {0};

    // Initialize Bluetooth communication
    server_socket = socket(AF_BLUETOOTH, SOCK_STREAM, BTPROTO_RFCOMM);
    if (server_socket < 0) {
        perror("Socket creation failed");
        exit(1);
    }

    // Set server address
    server_address.rc_family = AF_BLUETOOTH;
	server_address.rc_bdaddr = *BDADDR_ANY;
    server_address.rc_channel = (uint8_t)1;  // Use channel 1

    // Bind the server socket to the specified address and port
    if (bind(server_socket, (struct sockaddr *) &server_address, sizeof(server_address)) < 0) {
        perror("Binding failed");
        close(server_socket);
        exit(1);
    }

    // Listen for incoming connections
    if (listen(server_socket, BACKLOG) < 0) {
        perror("Listening failed");
        close(server_socket);
        exit(1);
    }

    printf("Server is listening for incoming connections...\n");

    for(; ;){
        // Accept a client connection
        address_length = sizeof(client_address);
        client_socket = accept(server_socket, (struct sockaddr *) &client_address, &address_length);
        if (client_socket < 0) {
            perror("Accepting connection failed");
            close(server_socket);
            exit(1);
        }

        ba2str(&client_address.rc_bdaddr, buffer);
        printf("Client connected: %s\n", buffer);

        // Handle the client in a separate function
        handleClient(client_socket);

        printf("Client disconnected: %s\n", buffer);
    }

    close(server_socket); 
	return 0;
}

