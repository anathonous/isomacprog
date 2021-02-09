#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

//static char *iso_name = {"mac.iso"};
//cc -g -Wall isomacprog.c -o isomacprog

int main(int argc, char **argv)
{
  int fd, ret;
  unsigned char buf[2048 - 64];
  off_t lba;
  size_t buf_size = 2048 - 64;

  if (argc < 2) {
    fprintf(stderr, "No iso name assigned\n");
    exit(1);
  }

  char *iso_name = argv[1];

  fd = open(iso_name, O_RDWR);
  if (fd == -1)
    goto err_ex;
  if (lseek(fd, (off_t) 32768 + 2048 + 71, SEEK_SET) == -1)
    goto err_ex;
  ret = read(fd, buf, 4);
  if (ret == -1)
    goto err_ex;
  if (ret < 4) {
    fprintf(stderr, "Cannot read 4 bytes from %s\n", iso_name);
    exit(1);
  }
  lba = buf[0] | (buf[1] << 8) | (buf[2] << 16) | (buf[3] << 24);
  if (lseek(fd, lba * 2048 + 64, SEEK_SET) == -1)
    goto err_ex;
  memset(buf, 0, buf_size);
  ret = write(fd, buf, buf_size);
  if (ret == -1)
    goto err_ex;
  if (ret < buf_size) {
    fprintf(stderr, "Cannot write %d bytes to %s\n", (int) buf_size, iso_name);
    exit(1);
  }
  close(fd);
  printf("done\n");
  exit(0);
err_ex:;
  perror(iso_name);
  exit(1);
}

