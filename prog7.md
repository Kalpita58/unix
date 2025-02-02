#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <dirent.h>
#include <unistd.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>

void print_permissions(mode_t mode) {
    char perms[] = "----------";
    if (S_ISDIR(mode)) perms[0] = 'd';
    if (mode & S_IRUSR) perms[1] = 'r';
    if (mode & S_IWUSR) perms[2] = 'w';
    if (mode & S_IXUSR) perms[3] = 'x';
    if (mode & S_IRGRP) perms[4] = 'r';
    if (mode & S_IWGRP) perms[5] = 'w';
    if (mode & S_IXGRP) perms[6] = 'x';
    if (mode & S_IROTH) perms[7] = 'r';
    if (mode & S_IWOTH) perms[8] = 'w';
    if (mode & S_IXOTH) perms[9] = 'x';
    printf("%s ", perms);
}

void list_dir(const char *path) {
    DIR *dir = opendir(path);
    if (!dir) { perror("opendir"); exit(1); }

    struct dirent *entry;
    struct stat st;
    char fullpath[1024];

    while ((entry = readdir(dir))) {
        snprintf(fullpath, sizeof(fullpath), "%s/%s", path, entry->d_name);
        if (stat(fullpath, &st) == -1) { perror("stat"); continue; }

        printf("%lu ", st.st_ino);
        print_permissions(st.st_mode);
        printf("%ld %s %s %5ld ", st.st_nlink, getpwuid(st.st_uid)->pw_name, 
               getgrgid(st.st_gid)->gr_name, st.st_size);

        char timebuf[16];
        strftime(timebuf, sizeof(timebuf), "%b %d %H:%M", localtime(&st.st_mtime));
        printf("%s %s\n", timebuf, entry->d_name);
    }

    closedir(dir);
}

int main(int argc, char *argv[]) {
    list_dir(argc > 1 ? argv[1] : ".");
    return 0;
}
