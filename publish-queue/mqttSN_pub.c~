#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <string.h>
#include <stdint.h>
#include <unistd.h>
#include <signal.h>


#include "queue.h"
#include "mqtt-sn.h"

const char *client_id = "LINUX";
const char *topic_name = "hello/world";
const char *message_data = "hi";
time_t keep_alive = 30;
const char *mqtt_sn_host = "10.255.61.51";
const char *mqtt_sn_port = "1883";
uint16_t topic_id = 0;
uint8_t topic_id_type = MQTT_SN_TOPIC_TYPE_NORMAL;
int8_t qos = 0;
int8_t retain = TRUE;
int8_t debug = FALSE;

struct tailq_entry {
	char *value;

	/*
	 * This holds the pointers to the next and previous entries in
	 * the tail queue.
	 */
	TAILQ_ENTRY(tailq_entry) entries;
};


/*
 * Our tail queue requires a head, this is defined using the
 * TAILQ_HEAD macro.
 */
TAILQ_HEAD(, tailq_entry) my_tailq_head;


int main(int argc, char* argv[])
{
	int sock;
	
	/* Define a pointer to an item in the tail queue. */
	struct tailq_entry *item;

        /* In some cases we have to track a temporary item. */
        struct tailq_entry *tmp_item;

	/* Initialize the tail queue. */
	TAILQ_INIT(&my_tailq_head);

	//Create UDP packet
	sock = mqtt_sn_create_socket(mqtt_sn_host, mqtt_sn_port);

	if (sock)
	{
		// Connect to gateway
	        if (qos >= 0) 
		{
            		mqtt_sn_send_connect(sock, client_id, keep_alive);
            		mqtt_sn_recieve_connack(sock);
        	}

		if (topic_id) 
		{
		    // Use pre-defined topic ID
		    topic_id_type = MQTT_SN_TOPIC_TYPE_PREDEFINED;
		} 
		else if (strlen(topic_name) == 2) 
		{
		    // Convert the 2 character topic name into a 2 byte topic id
		    topic_id = (topic_name[0] << 8) + topic_name[1];
		    topic_id_type = MQTT_SN_TOPIC_TYPE_SHORT;
		} 
		else if (qos >= 0) 
		{
		    // Register the topic name
		    mqtt_sn_send_register(sock, topic_name);
		    topic_id = mqtt_sn_recieve_regack(sock);
		    topic_id_type = MQTT_SN_TOPIC_TYPE_NORMAL;
		}
		
		int count = 1;
		char strbuf[255];
		int sent = 1;

		while (TRUE)
		{
			if (sent == 0)
			{
				sent = 	mqtt_sn_send_publish(sock, topic_id, topic_id_type, "test", qos, FALSE);
				if (sent == 0)
				{
					sprintf(strbuf, "%04d", count);
					count++;
					//add to the queue
					item = malloc(sizeof(*item));
					if (item == NULL) {
						perror("malloc failed");
						exit(EXIT_FAILURE);
					}

					item->value = strdup(strbuf);
					TAILQ_INSERT_TAIL(&my_tailq_head, item, entries);

					printf("No connection, message %s stored.\n", item->value);
					
				}
				else if (sent == 1)
				{
					//tmp_item = malloc(sizeof(*tmp_item));

					while(sent == 1 && !TAILQ_EMPTY(&my_tailq_head))
					{
						tmp_item = malloc(sizeof(*tmp_item));
						tmp_item = TAILQ_FIRST(&my_tailq_head);
						sent = mqtt_sn_send_publish(sock, topic_id, topic_id_type, tmp_item->value, qos, retain);
						
						if (sent == 1)
						{
							//If message was sent remove the message
							fprintf(stderr, "Stored Message %s Successfully Sent\n", tmp_item->value);
							TAILQ_REMOVE(&my_tailq_head, tmp_item, entries);
							free(tmp_item);
						}
					}	
				}

			}
			else
			{
				sprintf(strbuf, "%04d", count);
				count++;
				// Publish to the topic
				sent = mqtt_sn_send_publish(sock, topic_id, topic_id_type, strbuf, qos, retain);
				if (sent == 1)
				{
					fprintf(stderr, "Published we did\n");
				}
				else if (sent == 0)
				{
					//add to the queue
					item = malloc(sizeof(*item));
					if (item == NULL) {
						perror("malloc failed");
						exit(EXIT_FAILURE);
					}
					item->value = strdup(strbuf);
					TAILQ_INSERT_TAIL(&my_tailq_head, item, entries);

					fprintf(stderr, "No connection, message %s stored.\n", item->value);
				}
			}
			sleep(5);
		}

	}
	
	return 0;
	
}
