---
description: A demo giftcardreader application was given. here is how I discovered the bugs
---

# Basic Bugs in Demo Application

2 crashes and 1 infinite loop discovery:

Bug description:

crash1.gft => I discovered a **crash** by putting data length = -1 in gengift.py. This crashes the code at line at 198 where it tries to allocate memory. Since, program can't allocate negative memory, I fixed it by adding a check for negative number of bytes. Program exits if it detects negative number of bytes.

crash2.gft => type of record 3 in the gengift file **crashes** code at line 254 and then at line 260 because then it tries to allocate memory that is out of bounds. Non-animated gift card allocates a memory lesser than 256 bytes and animated gift card allocates 256 bytes plus. So, I added a check, if number of bytes are more than 255, it will only then execute the part where animated gift card is processed by pointers. Or else exits.

hang.gft => I discovered **an infinite loop** in genanim.py -> program data section by refering the lecture and adding \x09\xfd + 254 bytes of B to the data buffer will cause an infinite loop. I fixed it by going to giftcardreader.c in the animate() function and in case of "09" opcode, changed it to unsigned char from original char

### **Initial giftcardreader.c code:**

{% code lineNumbers="true" %}
```
/*
 * Gift Card Reading Application
 * Original Author: Shoddycorp's Cut-Rate Contracting
 * Comments added by: Justin Cappos (JAC) and Brendan Dolan-Gavitt (BDG)
 * Maintainer:
 * Date: 8 July 2020
 */


#include "giftcard.h"

#include <stdio.h>
#include <string.h>
#include <unistd.h>

// .,~==== interpreter for THX-1138 assembly ====~,.
//
// This is an emulated version of a microcontroller with
// 16 registers, one flag (the zero flag), and display
// functionality. Programs can operate on the message
// buffer and use opcode 0x07 to update the display, so
// that animated greetings can be created.
void animate(char *msg, unsigned char *program) {
    unsigned char regs[16];
    char *mptr = msg; // TODO: how big is this buffer?
    unsigned char *pc = program;
    int i = 0;
    int zf = 0;
    while (pc < program+256) {
        unsigned char op, arg1, arg2;
        op = *pc;
        arg1 = *(pc+1);
        arg2 = *(pc+2);
        switch (*pc) {
            case 0x00:
                break;
            case 0x01:
                regs[arg1] = *mptr;
                break;
            case 0x02:
                *mptr = regs[arg1];
                break;
            case 0x03:
                mptr += (char)arg1;
                break;
            case 0x04:
                regs[arg2] = arg1;
                break;
            case 0x05:
                regs[arg1] ^= regs[arg2];
                zf = !regs[arg1];
                break;
            case 0x06:
                regs[arg1] += regs[arg2];
                zf = !regs[arg1];
                break;
            case 0x07:
                puts(msg);
                break;
            case 0x08:
                goto done;
            case 0x09:
                pc += (char)arg1;
                break;
            case 0x10:
                if (zf) pc += (char)arg1;
                break;
        }
        pc+=3;
#ifndef FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION
        // Slow down animation to make it more visible (disabled if fuzzing)
        usleep(5000);
#endif
    }
done:
    return;
}

int get_gift_card_value(struct this_gift_card *thisone) {
	struct gift_card_data *gcd_ptr;
	struct gift_card_record_data *gcrd_ptr;
	struct gift_card_amount_change *gcac_ptr;
	int ret_count = 0;

	gcd_ptr = thisone->gift_card_data;
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
  		gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
		if (gcrd_ptr->type_of_record == 1) {
			gcac_ptr = gcrd_ptr->actual_record;
			ret_count += gcac_ptr->amount_added;
		}
	}
	return ret_count;
}

void print_gift_card_info(struct this_gift_card *thisone) {
	struct gift_card_data *gcd_ptr;
	struct gift_card_record_data *gcrd_ptr;
	struct gift_card_amount_change *gcac_ptr;
    struct gift_card_program *gcp_ptr;

	gcd_ptr = thisone->gift_card_data;
	printf("   Merchant ID: %32.32s\n",gcd_ptr->merchant_id);
	printf("   Customer ID: %32.32s\n",gcd_ptr->customer_id);
	printf("   Num records: %d\n",gcd_ptr->number_of_gift_card_records);
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
  		gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
		if (gcrd_ptr->type_of_record == 1) {
			printf("      record_type: amount_change\n");
			gcac_ptr = gcrd_ptr->actual_record;
			printf("      amount_added: %d\n",gcac_ptr->amount_added);
			if (gcac_ptr->amount_added>0) {
				printf("      signature: %32.32s\n",gcac_ptr->actual_signature);
			}
		}
		else if (gcrd_ptr->type_of_record == 2) {
			printf("      record_type: message\n");
			printf("      message: %s\n",(char *)gcrd_ptr->actual_record);
		}
		else if (gcrd_ptr->type_of_record == 3) {
            gcp_ptr = gcrd_ptr->actual_record;
			printf("      record_type: animated message\n");
            // BDG: Hmm... is message guaranteed to be null-terminated?
            printf("      message: %s\n", gcp_ptr->message);
            printf("  [running embedded program]  \n");
            animate(gcp_ptr->message, gcp_ptr->program);
		}
	}
	printf("  Total value: %d\n\n",get_gift_card_value(thisone));
}

// Added to support web functionalities
void gift_card_json(struct this_gift_card *thisone) {
    struct gift_card_data *gcd_ptr;
    struct gift_card_record_data *gcrd_ptr;
    struct gift_card_amount_change *gcac_ptr;
    gcd_ptr = thisone->gift_card_data;
    printf("{\n");
    printf("  \"merchant_id\": \"%32.32s\",\n", gcd_ptr->merchant_id);
    printf("  \"customer_id\": \"%32.32s\",\n", gcd_ptr->customer_id);
    printf("  \"total_value\": %d,\n", get_gift_card_value(thisone));
    printf("  \"records\": [\n");
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
        gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
        printf("    {\n");
        if (gcrd_ptr->type_of_record == 1) {
            printf("      \"record_type\": \"amount_change\",\n");
            gcac_ptr = gcrd_ptr->actual_record;
            printf("      \"amount_added\": %d,\n",gcac_ptr->amount_added);
            if (gcac_ptr->amount_added>0) {
                printf("      \"signature\": \"%32.32s\"\n",gcac_ptr->actual_signature);
            }
        }
        else if (gcrd_ptr->type_of_record == 2) {
			printf("      \"record_type\": \"message\",\n");
			printf("      \"message\": \"%s\"\n",(char *)gcrd_ptr->actual_record);
        }
        else if (gcrd_ptr->type_of_record == 3) {
            struct gift_card_program *gcp = gcrd_ptr->actual_record;
			printf("      \"record_type\": \"animated message\",\n");
			printf("      \"message\": \"%s\",\n",gcp->message);
            // programs are binary so we will hex for the json
            char *hexchars = "01234567890abcdef";
            char program_hex[512+1];
            program_hex[512] = '\0';
            int i;
            for(i = 0; i < 256; i++) {
                program_hex[i*2] = hexchars[((gcp->program[i] & 0xf0) >> 4)];
                program_hex[i*2+1] = hexchars[(gcp->program[i] & 0x0f)];
            }
			printf("      \"program\": \"%s\"\n",program_hex);
        }
        if (i < gcd_ptr->number_of_gift_card_records-1)
            printf("    },\n");
        else
            printf("    }\n");
    }
    printf("  ]\n");
    printf("}\n");
}


struct this_gift_card *gift_card_reader(FILE *input_fd) {

	struct this_gift_card *ret_val = malloc(sizeof(struct this_gift_card));

    void *optr;
	void *ptr;

	// Loop to do the whole file
	while (!feof(input_fd)) {

		struct gift_card_data *gcd_ptr;
		/* JAC: Why aren't return types checked? */
		fread(&ret_val->num_bytes, 4,1, input_fd);

		// Make something the size of the rest and read it in
		ptr = malloc(ret_val->num_bytes);
		fread(ptr, ret_val->num_bytes, 1, input_fd);

        optr = ptr-4;

		gcd_ptr = ret_val->gift_card_data = malloc(sizeof(struct gift_card_data));
		gcd_ptr->merchant_id = ptr;
		ptr += 32;
//		printf("VD: %d\n",(int)ptr - (int) gcd_ptr->merchant_id);
		gcd_ptr->customer_id = ptr;
		ptr += 32;
		/* JAC: Something seems off here... */
		gcd_ptr->number_of_gift_card_records = *((char *)ptr);
		ptr += 4;

		gcd_ptr->gift_card_record_data = (void *)malloc(gcd_ptr->number_of_gift_card_records*sizeof(void*));

		// Now ptr points at the gift card recrod data
		for (int i=0; i < gcd_ptr->number_of_gift_card_records; i++){
			//printf("i: %d\n",i);
			struct gift_card_record_data *gcrd_ptr;
			gcrd_ptr = gcd_ptr->gift_card_record_data[i] = malloc(sizeof(struct gift_card_record_data));
			struct gift_card_amount_change *gcac_ptr;
			gcac_ptr = gcrd_ptr->actual_record = malloc(sizeof(struct gift_card_record_data));
            struct gift_card_program *gcp_ptr;
			gcp_ptr = malloc(sizeof(struct gift_card_program));

			gcrd_ptr->record_size_in_bytes = *((char *)ptr);
            //printf("rec at %x, %d bytes\n", ptr - optr, gcrd_ptr->record_size_in_bytes);
			ptr += 4;
			//printf("record_data: %d\n",gcrd_ptr->record_size_in_bytes);
			gcrd_ptr->type_of_record = *((char *)ptr);
			ptr += 4;
            //printf("type of rec: %d\n", gcrd_ptr->type_of_record);

			// amount change
			if (gcrd_ptr->type_of_record == 1) {
				gcac_ptr->amount_added = *((int*) ptr);
				ptr += 4;

				// don't need a sig if negative
				/* JAC: something seems off here */
				if (gcac_ptr < 0) break;

				gcac_ptr->actual_signature = ptr;
				ptr+=32;
			}
			// message
			if (gcrd_ptr->type_of_record == 2) {
				gcrd_ptr->actual_record = ptr;
				// advance by the string size + 1 for nul
                // BDG: does not seem right
				ptr=ptr+strlen((char *)gcrd_ptr->actual_record)+1;
			}
            // BDG: gift cards can run code?? Might want to check this one carefully...
            // text animatino (BETA)
            if (gcrd_ptr->type_of_record == 3) {
                gcp_ptr->message = malloc(32);
                gcp_ptr->program = malloc(256);
                memcpy(gcp_ptr->message, ptr, 32);
                ptr+=32;
                memcpy(gcp_ptr->program, ptr, 256);
                ptr+=256;
                gcrd_ptr->actual_record = gcp_ptr;
            }

            if (gcrd_ptr->type_of_record > 3) {
                printf("unknown record type: %d\n", gcrd_ptr->type_of_record);
                exit(1);
            }
		}
	}
	return ret_val;
}

struct this_gift_card *thisone;

int main(int argc, char **argv) {
    if (argc != 3) {
        fprintf(stderr, "usage: %s <1|2> file.gft\n", argv[0]);
        fprintf(stderr, "  - Use 1 for text output, 2 for JSON output\n");
        return 1;
    }
	FILE *input_fd = fopen(argv[2],"r");
    if (!input_fd) {
        fprintf(stderr, "error opening file\n");
        return 1;
    }
	thisone = gift_card_reader(input_fd);
	if (argv[1][0] == '1') print_gift_card_info(thisone);
    else if (argv[1][0] == '2') gift_card_json(thisone);

	return 0;
}
```
{% endcode %}

### Initial gengift.py code:

To generate a giftcard file: python3 gengift.py name.gft

{% code lineNumbers="true" %}
```
import struct
import sys

# We build the content of the file in a byte string first
# This lets us calculate the length for the header at the end
data = b''
data += b"GiftCardz.com".ljust(32, b' ') # Merchant ID
data += b"B"*32 # Customer ID
data += struct.pack("<I", 1) # One record
# Record of type message
data += struct.pack("<I", 8 + 32)       # Record size: 4 bytes size, 4 bytes type, 32 bytes message
data += struct.pack("<I", 2)            # Record type
data += b"x"*31 + b'\0'                 # Note: 32 byte message

f = open(sys.argv[1], 'wb')
datalen = len(data) + 4 # Plus 4 bytes for the length itself
f.write(struct.pack("<I", datalen))
f.write(data)
f.close()
```
{% endcode %}

### Initial genanim.py code:

To generate animated giftcard file: python3 genanim.py animated\_name.gft

{% code lineNumbers="true" %}
```
import struct
import sys

# We build the content of the file in a byte string first
# This lets us calculate the length for the header at the end
data = b''
data += b"A"*32 # Merchant ID
data += b"B"*32 # Customer ID
data += struct.pack("<I", 1) # One record
# Record of type animation
data += struct.pack("<I", 8 + 32 + 256) # Record size (4 bytes)
data += struct.pack("<I", 3)            # Record type (4 bytes)
data += b"A"*31 + b'\x00'               # Note: 32 byte message
data += b'\x08' * 256                   # Program made entirely of "end program" (256 bytes)

f = open(sys.argv[1], 'wb')
datalen = len(data) + 4 # Plus 4 bytes for the length itself
f.write(struct.pack("<I", datalen))
f.write(data)
f.close()
```
{% endcode %}

### Crash 1 python file:

{% code lineNumbers="true" %}
```
import struct
import sys

# We build the content of the file in a byte string first
# This lets us calculate the length for the header at the end
data = b''
data += b"GiftCardz.com".ljust(32, b' ') # Merchant ID
data += b"B"*32 # Customer ID
data += struct.pack("<I", 1) # One record
# Record of type message
data += struct.pack("<I", 8 + 32)       # Record size: 4 bytes size, 4 bytes type, 32 bytes message
data += struct.pack("<I", 1)            # Record type
data += b"x"*31 + b'\0'                 # Note: 32 byte message

f = open(sys.argv[1], 'wb')
datalen = -1		 		# crash for -1 data length
f.write(struct.pack("<i", datalen))
f.write(data)
f.close()
```
{% endcode %}

### Crash 2 Python file

{% code lineNumbers="true" %}
```
import struct
import sys

# We build the content of the file in a byte string first
# This lets us calculate the length for the header at the end
data = b''
data += b"GiftCardz.com".ljust(32, b' ') # Merchant ID
data += b"B"*32 # Customer ID
data += struct.pack("<I", 1) # One record
# Record of type message
data += struct.pack("<I", 8 + 32)       # Record size: 4 bytes size, 4 bytes type, 32 bytes message
data += struct.pack("<I", 3)            # Record type
data += b"x"*31 + b'\0'                 # Note: 32 byte message

f = open(sys.argv[1], 'wb')
datalen = len(data) + 4 # Plus 4 bytes for the length itself
f.write(struct.pack("<I", datalen))
f.write(data)
f.close()
```
{% endcode %}

### Hang file python code:

{% code lineNumbers="true" %}
```
import struct
import sys

# We build the content of the file in a byte string first
# This lets us calculate the length for the header at the end
data = b''
data += b"A"*32 # Merchant ID
data += b"B"*32 # Customer ID
data += struct.pack("<I", 1) # One record
# Record of type animation
data += struct.pack("<I", 8 + 32 + 256) # Record size (4 bytes)
data += struct.pack("<I", 3)            # Record type (4 bytes)
data += b"A"*31 + b'\x00'               # Note: 32 byte message
data += b'\x09\xfd' +b'\x08'* 254       # Program to make it loop infinitely and then random 254 bytes

f = open(sys.argv[1], 'wb')
datalen = len(data) + 4 # Plus 4 bytes for the length itself
f.write(struct.pack("<I", datalen))
f.write(data)
f.close()
```
{% endcode %}

### Fixed C code:

{% code lineNumbers="true" %}
```
/*
 * Gift Card Reading Application
 * Original Author: Shoddycorp's Cut-Rate Contracting
 * Comments added by: Justin Cappos (JAC) and Brendan Dolan-Gavitt (BDG)
 * Maintainer:
 * Date: 8 July 2020
 */


#include "giftcard.h"

#include <stdio.h>
#include <string.h>
#include <unistd.h>

// .,~==== interpreter for THX-1138 assembly ====~,.
//
// This is an emulated version of a microcontroller with
// 16 registers, one flag (the zero flag), and display
// functionality. Programs can operate on the message
// buffer and use opcode 0x07 to update the display, so
// that animated greetings can be created.
void animate(char *msg, unsigned char *program) {
    unsigned char regs[16];
    char *mptr = msg; // TODO: how big is this buffer?
    unsigned char *pc = program;
    int i = 0;
    int zf = 0;
    while (pc < program+256) {
        unsigned char op, arg1, arg2;
        op = *pc;
        arg1 = *(pc+1);
        arg2 = *(pc+2);
        switch (*pc) {
            case 0x00:
                break;
            case 0x01:
                regs[arg1] = *mptr;
                break;
            case 0x02:
                *mptr = regs[arg1];
                break;
            case 0x03:
                mptr += (char)arg1;
                break;
            case 0x04:
                regs[arg2] = arg1;
                break;
            case 0x05:
                regs[arg1] ^= regs[arg2];
                zf = !regs[arg1];
                break;
            case 0x06:
                regs[arg1] += regs[arg2];
                zf = !regs[arg1];
                break;
            case 0x07:
                puts(msg);
                break;
            case 0x08:
                goto done;
            case 0x09:
                pc += (unsigned char)arg1; /* fix 3 for hang (infinite loop) */
                break;
            case 0x10:
                if (zf) pc += (char)arg1;
                break;
        }
        pc+=3;
#ifndef FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION
        // Slow down animation to make it more visible (disabled if fuzzing)
        usleep(5000);
#endif
    }
done:
    return;
}

int get_gift_card_value(struct this_gift_card *thisone) {
	struct gift_card_data *gcd_ptr;
	struct gift_card_record_data *gcrd_ptr;
	struct gift_card_amount_change *gcac_ptr;
	int ret_count = 0;

	gcd_ptr = thisone->gift_card_data;
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
  		gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
		if (gcrd_ptr->type_of_record == 1) {
			gcac_ptr = gcrd_ptr->actual_record;
			ret_count += gcac_ptr->amount_added;
		}
	}
	return ret_count;
}

void print_gift_card_info(struct this_gift_card *thisone) {
	struct gift_card_data *gcd_ptr;
	struct gift_card_record_data *gcrd_ptr;
	struct gift_card_amount_change *gcac_ptr;
    struct gift_card_program *gcp_ptr;

	gcd_ptr = thisone->gift_card_data;
	printf("   Merchant ID: %32.32s\n",gcd_ptr->merchant_id);
	printf("   Customer ID: %32.32s\n",gcd_ptr->customer_id);
	printf("   Num records: %d\n",gcd_ptr->number_of_gift_card_records);
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
  		gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
		if (gcrd_ptr->type_of_record == 1) {
			printf("      record_type: amount_change\n");
			gcac_ptr = gcrd_ptr->actual_record;
			printf("      amount_added: %d\n",gcac_ptr->amount_added);
			if (gcac_ptr->amount_added>0) {
				printf("      signature: %32.32s\n",gcac_ptr->actual_signature);
			}
		}
		else if (gcrd_ptr->type_of_record == 2) {
			printf("      record_type: message\n");
			printf("      message: %s\n",(char *)gcrd_ptr->actual_record);
		}
		else if (gcrd_ptr->type_of_record == 3) {
            gcp_ptr = gcrd_ptr->actual_record;
			printf("      record_type: animated message\n");
            // BDG: Hmm... is message guaranteed to be null-terminated?
            printf("      message: %s\n", gcp_ptr->message);
            printf("  [running embedded program]  \n");
            animate(gcp_ptr->message, gcp_ptr->program);
		}
	}
	printf("  Total value: %d\n\n",get_gift_card_value(thisone));
}

// Added to support web functionalities
void gift_card_json(struct this_gift_card *thisone) {
    struct gift_card_data *gcd_ptr;
    struct gift_card_record_data *gcrd_ptr;
    struct gift_card_amount_change *gcac_ptr;
    gcd_ptr = thisone->gift_card_data;
    printf("{\n");
    printf("  \"merchant_id\": \"%32.32s\",\n", gcd_ptr->merchant_id);
    printf("  \"customer_id\": \"%32.32s\",\n", gcd_ptr->customer_id);
    printf("  \"total_value\": %d,\n", get_gift_card_value(thisone));
    printf("  \"records\": [\n");
	for(int i=0;i<gcd_ptr->number_of_gift_card_records; i++) {
        gcrd_ptr = (struct gift_card_record_data *) gcd_ptr->gift_card_record_data[i];
        printf("    {\n");
        if (gcrd_ptr->type_of_record == 1) {
            printf("      \"record_type\": \"amount_change\",\n");
            gcac_ptr = gcrd_ptr->actual_record;
            printf("      \"amount_added\": %d,\n",gcac_ptr->amount_added);
            if (gcac_ptr->amount_added>0) {
                printf("      \"signature\": \"%32.32s\"\n",gcac_ptr->actual_signature);
            }
        }
        else if (gcrd_ptr->type_of_record == 2) {
			printf("      \"record_type\": \"message\",\n");
			printf("      \"message\": \"%s\"\n",(char *)gcrd_ptr->actual_record);
        }
        else if (gcrd_ptr->type_of_record == 3) {
            struct gift_card_program *gcp = gcrd_ptr->actual_record;
			printf("      \"record_type\": \"animated message\",\n");
			printf("      \"message\": \"%s\",\n",gcp->message);
            // programs are binary so we will hex for the json
            char *hexchars = "01234567890abcdef";
            char program_hex[512+1];
            program_hex[512] = '\0';
            int i;
            for(i = 0; i < 256; i++) {
                program_hex[i*2] = hexchars[((gcp->program[i] & 0xf0) >> 4)];
                program_hex[i*2+1] = hexchars[(gcp->program[i] & 0x0f)];
            }
			printf("      \"program\": \"%s\"\n",program_hex);
        }
        if (i < gcd_ptr->number_of_gift_card_records-1)
            printf("    },\n");
        else
            printf("    }\n");
    }
    printf("  ]\n");
    printf("}\n");
}


struct this_gift_card *gift_card_reader(FILE *input_fd) {

	struct this_gift_card *ret_val = malloc(sizeof(struct this_gift_card));

    void *optr;
	void *ptr;

	// Loop to do the whole file
	while (!feof(input_fd)) {

		struct gift_card_data *gcd_ptr;
		/* JAC: Why aren't return types checked? */
		fread(&ret_val->num_bytes, 4,1, input_fd);

		/* fix for crash 1 -> number bytes = -1 */

		if (ret_val->num_bytes<0){ //if statement to fix the crashes
			printf("Invalid byte value! Exiting now...");
			printf("\n");
			exit(0);
		}

		// Make something the size of the rest and read it in
		ptr = malloc(ret_val->num_bytes);
		fread(ptr, ret_val->num_bytes, 1, input_fd);

        optr = ptr-4;

		gcd_ptr = ret_val->gift_card_data = malloc(sizeof(struct gift_card_data));
		gcd_ptr->merchant_id = ptr;
		ptr += 32;
//		printf("VD: %d\n",(int)ptr - (int) gcd_ptr->merchant_id);
		gcd_ptr->customer_id = ptr;
		ptr += 32;
		/* JAC: Something seems off here... */
		gcd_ptr->number_of_gift_card_records = *((int *)ptr);
		ptr += 4;

		gcd_ptr->gift_card_record_data = (void *)malloc(gcd_ptr->number_of_gift_card_records*sizeof(void*));

		// Now ptr points at the gift card recrod data
		for (int i=0; i < gcd_ptr->number_of_gift_card_records; i++){
			//printf("i: %d\n",i);
			struct gift_card_record_data *gcrd_ptr;
			gcrd_ptr = gcd_ptr->gift_card_record_data[i] = malloc(sizeof(struct gift_card_record_data));
			struct gift_card_amount_change *gcac_ptr;
			gcac_ptr = gcrd_ptr->actual_record = malloc(sizeof(struct gift_card_record_data));
            struct gift_card_program *gcp_ptr;
			gcp_ptr = malloc(sizeof(struct gift_card_program));

			gcrd_ptr->record_size_in_bytes = *((char *)ptr);
            //printf("rec at %x, %d bytes\n", ptr - optr, gcrd_ptr->record_size_in_bytes);
			ptr += 4;
			//printf("record_data: %d\n",gcrd_ptr->record_size_in_bytes);
			gcrd_ptr->type_of_record = *((char *)ptr);
			ptr += 4;
            //printf("type of rec: %d\n", gcrd_ptr->type_of_record);

			// amount change
			if (gcrd_ptr->type_of_record == 1) {
				gcac_ptr->amount_added = *((int*) ptr);
				ptr += 4;

				// don't need a sig if negative
				/* JAC: something seems off here */
				if (gcac_ptr < 0) break;

				gcac_ptr->actual_signature = ptr;
				ptr+=32;
			}
			// message
			if (gcrd_ptr->type_of_record == 2) {
				gcrd_ptr->actual_record = ptr;
				// advance by the string size + 1 for nul
                // BDG: does not seem right
				ptr=ptr+strlen((char *)gcrd_ptr->actual_record)+1;
			}
            // BDG: gift cards can run code?? Might want to check this one carefully...
            // text animatino (BETA)
            
            /* fix for crash 2 */
            
            if (gcrd_ptr->type_of_record == 3 && ret_val->num_bytes > 255 ) {
                gcp_ptr->message = malloc(32);
                gcp_ptr->program = malloc(256);
                memcpy(gcp_ptr->message, ptr, 32);
                ptr+=32;
                memcpy(gcp_ptr->program, ptr, 256);
                ptr+=256;
                gcrd_ptr->actual_record = gcp_ptr;
            }
            else{
            	printf("Invalid record type for the specified gift card! Exiting...\n");
            	exit(1);
            }

            if (gcrd_ptr->type_of_record > 3) {
                printf("unknown record type: %d\n", gcrd_ptr->type_of_record);
                exit(1);
            }
		}
	}
	return ret_val;
}

struct this_gift_card *thisone;

int main(int argc, char **argv) {
    if (argc != 3) {
        fprintf(stderr, "usage: %s <1|2> file.gft\n", argv[0]);
        fprintf(stderr, "  - Use 1 for text output, 2 for JSON output\n");
        return 1;
    }
	FILE *input_fd = fopen(argv[2],"r");
    if (!input_fd) {
        fprintf(stderr, "error opening file\n");
        return 1;
    }
	thisone = gift_card_reader(input_fd);
	if (argv[1][0] == '1') print_gift_card_info(thisone);
    else if (argv[1][0] == '2') gift_card_json(thisone);

	return 0;
}
```
{% endcode %}
