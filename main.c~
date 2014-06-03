#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mysql/mysql.h>

#define MIN3(a, b, c) ((a) < (b) ? ((a) < (c) ? (a) : (c)) : ((b) < (c) ? (b) : (c)))
#define ID_LEN 5

typedef struct {
	char *firstname;
	char *lastname;
	char *email;
	int userpk;
} DATA;

DATA 	*the_array = NULL;
int 	 num_elements = 0; // To keep track of the number of elements used
int		num_allocated = 0; // This is essentially how large the array is

int levenshtein(char *s1, char *s2) {
	unsigned long x, y, s1len, s2len;
	s1len = strlen(s1);
	s2len = strlen(s2);
	unsigned long matrix[s2len+1][s1len+1];
	matrix[0][0] = 0;
	for (x = 1; x <= s2len; x++){
		matrix[x][0] = matrix[x-1][0] + 1;
	}
	for (y = 1; y <= s1len; y++){
		matrix[0][y] = matrix[0][y-1] + 1;
	}
	for (x = 1; x <= s2len; x++){
		for (y = 1; y <= s1len; y++){
			matrix[x][y] = MIN3(matrix[x-1][y] + 1, matrix[x][y-1] + 1, matrix[x-1][y-1] + (s1[y-1] == s2[x-1] ? 0 : 1));
		}
	}

	return(matrix[s2len][s1len]);
}

int AddToArray (DATA item){
	if(num_elements == num_allocated) { // Are more refs required?
		// Feel free to change the initial number of refs and the rate at which refs are allocated.
		if (num_allocated == 0)
			num_allocated = 30; // Start off with 3 refs
		else
			num_allocated *= 20; // Double the number of refs allocated

		// Make the reallocation transactional by using a temporary variable first
		DATA *_tmp = realloc(the_array, (num_allocated * sizeof(DATA)));

		// If the reallocation didn't go so well, inform the user and bail out
		if (!_tmp)
		{
			fprintf(stderr, "ERROR: Couldn't realloc memory!\n");
			return -1 ;
		}

		// Things are looking good so far, so let's set the
		the_array = (DATA*)_tmp;
	}

	the_array[num_elements] = item;
	num_elements++;

	return num_elements;
}

int main(){
	char *firstname1;
	char *firstname2;
	char *lastname1;
	char *lastname2;
	char *email1;
	char *email2;

	int index1 = 0;
	int index2 = 0;
	int pk1 = 0;
	int pk2 = 0;

	DATA temp;

	MYSQL *conn;
	MYSQL_RES *res;
	MYSQL_ROW row;
	char *server = "10.49.0.16";
	char *user = "stba";
	char *password = "UB2kiML8"; /* set me first */
	char *database = "portal_content";
	//Todo dynamic number
	int  i = 0;
	int  z, y, x;

	conn = mysql_init(NULL);
	/* Connect to database */
	if (!mysql_real_connect(conn, server,
                            user, password, database, 0, NULL, 0)) {
		fprintf(stderr, "%s\n", mysql_error(conn));
		return 1;
	}
	/* send SQL query */
	//Todo dynamic number
	if (mysql_query(conn, "SELECT userpk, lastname, firstname, email FROM user_data WHERE itime > '2012' ORDER BY firstname, lastname ASC LIMIT 10000")) {
		fprintf(stderr, "%s\n", mysql_error(conn));
		return 1;
	}

	res = mysql_use_result(conn);

	while ((row = mysql_fetch_row(res)) != NULL){

		if (row[1] != NULL && row[2] != NULL && row[3] != NULL){

			temp.firstname = malloc((strlen(row[2]) + 1));
			strncpy(temp.firstname, row[2], strlen(row[2]) + 1);

			temp.lastname = malloc((strlen(row[1]) + 1));
			strncpy(temp.lastname, row[1], strlen(row[1]) + 1);

			temp.email = malloc((strlen(row[3]) + 1));
			strncpy(temp.email, row[3], strlen(row[3]) + 1);

			temp.userpk = atoi(row[0]);

			if (AddToArray(temp) == -1){
				return 1;				// kamikaze
			}

			printf("Fetching Results: %d\n",i);
		
			i++;
		}
	}

	/* close connection */
	mysql_free_result(res);
	mysql_close(conn);

	int arrSize = i;
	

	unsigned long totalLevenshtein=0;
	unsigned long tempTotalLevenshtein=0;
	float averageLevenshtein=0;
	unsigned long lev=0;
	int tempLev1, tempLev2;
	int isDupe = 0;
	static volatile float percentage;
	
	printf("%d Results\n\n",arrSize);

	getchar();
	
	// Open a file and ...
	FILE *out;
	out = fopen("output.csv", "w");

	for (z = 0; z < i-1; z++)
	{
		
		
		index1 = z;
		index2 = z+1;
		
		firstname1=the_array[index1].firstname;
		firstname2=the_array[index2].firstname;

		lastname1=the_array[index1].lastname;
		lastname2=the_array[index2].lastname;

		pk1=the_array[index1].userpk;
		pk2=the_array[index2].userpk;

		email1=the_array[index1].email;
		email2=the_array[index2].email;
		
		y = 1;
		isDupe = 0;	
		
		for (x = 0; x <= arrSize-1 || isDupe ==1 ; x++){
			
		
			firstname1=the_array[index1].firstname;
			firstname2=the_array[x].firstname;

			lastname1=the_array[index1].lastname;
			lastname2=the_array[x].lastname;

			email1=the_array[index1].email;
			email2=the_array[x].email;
			
			tempLev1 = levenshtein(firstname1,firstname2);
			tempLev2 = levenshtein(lastname1,lastname2);
			
			//here's the wurm
			if(strcmp(email1,email2)==0){
				isDupe = 1;
			}else if(tempLev1 < 3 && tempLev2 < 3){
				tempTotalLevenshtein = tempTotalLevenshtein + tempLev1 + tempLev2;
				y++;
				y++;
			}
		}
		
		averageLevenshtein = tempTotalLevenshtein / y;
		
		
		tempTotalLevenshtein=0;
		
		fprintf(out, "%d;%s;%s;%s;%f;%d\n", pk1, firstname1, lastname1, email1, averageLevenshtein, isDupe);
		
		percentage = (((float)z / (float)arrSize) * (float)100);
		//printf("%d / %d\t\t%3.2f%%\n ",z,arrSize,percentage);
		//printf("%d    -  ",isDupe);
		
		//This for the human readable output
		if(isDupe == 1 || averageLevenshtein<1){
  			printf("%c[1;32m%d    -   %3.2f%% Done - %d %s %s %s Average Levenshtein distance: %3.2f", 27, isDupe,percentage, pk1, firstname1, lastname1, email1, averageLevenshtein); // red
		
		}else{
  			printf("%c[1;0m%d    -   %c[1;31m%3.2f%% Done - %d %s %s %s Average Levenshtein distance: %3.2f", 27, isDupe, 27, percentage, pk1, firstname1, lastname1, email1, averageLevenshtein); // green
		}
		
		printf("%c[1;0m\n",27);
		
		if(isDupe == 1){
				getchar();
		}
	}
	
	totalLevenshtein = totalLevenshtein/z;
	
	fclose(out);

	printf("Total Levenshtein after 1st iteration %lu\n\n",totalLevenshtein);

	return 0;
}
