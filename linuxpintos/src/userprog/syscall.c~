#include "userprog/syscall.h"
#include <stdio.h>
#include <syscall-nr.h>
#include "threads/interrupt.h"
#include "threads/thread.h"

#include "threads/init.h"
#include "filesys/filesys.h"
#include "filesys/file.h"
#include "userprog/process.h"

static void syscall_handler (struct intr_frame *);

void syscall_init (void){
  intr_register_int (0x30, 3, INTR_ON, syscall_handler, "syscall");
}
static void close_by_fd(size_t fd,struct thread * thrd){
	bitmap_flip(thrd->bitmap, fd);
	file_close(thrd->files[fd-2]);
	
}

static void syscall_handler (struct intr_frame *f UNUSED){
	struct thread * thrd=thread_current();
  //printf ("system call!\n");
  //thread_exit ();
	int *p=f->esp;
	switch(*p){
	case SYS_EXEC:{
		char * cmd_line=(char *)(*(p+1));
		tid_t child_process = process_execute(cmd_line); //tries to create a new thread
		if(child_process!=TID_ERROR){
			f->eax=child_process;
		}
		else{
			f->eax=-1;
		}
	break;
		
	}
	case SYS_HALT:
		power_off();
		break;
	case SYS_CREATE:{
		const char *first_arg=(char *)(*(p+1));
		int second_arg=*(p+2);
		f->eax=filesys_create(first_arg,second_arg);		
		break;
	}
	case SYS_WRITE:{
		int fd=*(p+1);
		char* text =  (char *)(*(p+2));
		int buffsize=*(p+3);
		if(fd==STDOUT_FILENO){
			putbuf(text,buffsize);
			f->eax=buffsize;	
		}
		else{
			if(fd>=2 && fd < 129 && bitmap_test(thrd->bitmap,fd)){
			f->eax= file_write(thrd->files[fd-2],text,buffsize);		
			}
			else f->eax=-1;
		}
		break;
	}
	case SYS_READ:{
		int fd=*(p+1);
		char* text =  (char*)(*(p+2));
                int buffsize=*(p+3);
                if(fd==STDIN_FILENO){
			int count = 0;
			int size=buffsize;
			while(count<size){
				text[count]=input_getc();
				count++;			
			}
			f->eax=buffsize;
                }
                else{
			if(fd > 1 && fd < 129 && bitmap_test(thrd->bitmap,fd)){
			
                        f->eax=file_read(thrd->files[fd-2],text,buffsize);
			}
			else f->eax=-1;
                }
                break;

		}
	case SYS_OPEN:{
		char *name= (char*)(*(p+1));
		struct file * open_file  =filesys_open(name);
		if(open_file != NULL){
			size_t fd=bitmap_scan_and_flip(thrd->bitmap,2,1,0);
			if(fd == BITMAP_ERROR){	//no more space for files
				file_close(open_file);
				f->eax = -1;
			}
			else{
				thrd->files[fd-2]=open_file;
				f->eax=fd;
			}		
		}
		else{
			f->eax=-1;
		}
		break;
		}
	case SYS_EXIT:{
		thread_exit();
		break;
		}
	case SYS_CLOSE:{ 
		size_t fd = *(p+1);
		if(bitmap_test(thrd->bitmap,fd)) close_by_fd(fd,thrd);
		break;
		}
	default:
		//printf(*p);
		printf("SYS_CALL: 404");	
		thread_exit();
	

	}
	//shutdown the machine

	//WRITE:
	//get parameters...
	//call the right functions
}
