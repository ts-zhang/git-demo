# libgit2 api使用

libgit2 is a portable, pure C implementation of the Git core methods provided as a re-entrant linkable library with a solid API, allowing you to write native speed custom Git applications in any language which supports C bindings.

## 初始化仓库
```c
int init_repo() {
    git_libgit2_init();
    git_repository *repo = NULL;
    int ret = git_repository_init(&repo, "../repo/1", 0);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("init error: %s\n", err->message);
    }
    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## 初始化裸仓库
```c
int init_bare_repo() {
    git_libgit2_init();
    git_repository *repo = NULL;
    int ret = git_repository_init(&repo, "../repo/2", 1);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("init error: %s\n", err->message);
    }
    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## 初始化仓库带描述信息
```c
int init_repo_with_desc() {
    int ret;
    git_repository *repo = NULL;
    git_repository_init_options opts = GIT_REPOSITORY_INIT_OPTIONS_INIT;
    opts.flags |= GIT_REPOSITORY_INIT_MKPATH;
    opts.description = "my repo with desc\n";
    git_libgit2_init();
    ret = git_repository_init_ext(&repo, "../repo/3", &opts);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("init error: %s\n", err->message);
    }
    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## 克隆仓库
```c
int clone_repo() {
    git_libgit2_init();
    git_repository *repo = NULL;
    const char *repo_url = "https://github.com/libgit2/libgit2.git";
    const char *repo_path = "../repo/libgit2";
    int ret = git_clone(&repo, repo_url, repo_path, NULL);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("clone error: %s\n", err->message);
    }
    git_libgit2_shutdown();
    return ret;
}
```

## 克隆仓库(带进度)
定义两个回调函数，checkout回调和fetch回调

```c
/**
 * checkout 本地库应用到工作区进度
 * @param path
 * @param cur
 * @param tot
 * @param payload
 */
void clone_checkout_progress_callback(const char *path, size_t cur, size_t tot, void *payload) {
    printf("总步骤：%u，已完成：%u，文件：%s\n", tot, cur, path);
}
```
```c
/**
 * fetch 远程数据取回 网络下载进度
 * @param stats
 * @param payload
 * @return
 */
int fetch_progress(const git_transfer_progress *stats, void *payload) {
    size_t kbytes = stats->received_bytes / 1024;
    printf("总对象数：%d，已下载对象数：%d，已索引对象数：%d，已接收字节数：%5uKB\n", stats->total_objects, stats->received_objects,
           stats->indexed_objects, kbytes);
    return 0;
}
```
```c
int clone_repo_with_progress() {
    git_libgit2_init();
    git_repository *repo = NULL;
    const char *repo_url = "https://github.com/libgit2/libgit2.git";
    const char *repo_path = "../repo/libgit2";

    git_clone_options clone_opts = GIT_CLONE_OPTIONS_INIT;
    clone_opts.checkout_opts.checkout_strategy = GIT_CHECKOUT_SAFE;
    clone_opts.checkout_opts.progress_cb = clone_checkout_progress_callback;
    clone_opts.fetch_opts.callbacks.transfer_progress = fetch_progress;

    int ret = git_clone(&repo, repo_url, repo_path, &clone_opts);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("clone error: %s\n", err->message);
    }
    git_libgit2_shutdown();
    return ret;
}
```

## 打开仓库
```c
int open_repo() {
    const char *repo_path = "../repo/libgit2";
    git_libgit2_init();
    git_repository *repo = NULL;
    int ret = git_repository_open(&repo, repo_path);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("clone error: %s\n", err->message);
    }
    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## 查看目录是否包含仓库
```c
int find_repo() {
    git_libgit2_init();
    const char *repo_root_path = "../repo/libgit2";
    git_buf root = {0};
    int ret = git_repository_discover(&root, repo_root_path, 0, NULL);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("discover error: %s\n", err->message);
    } else {
        printf("discover repo: %s\n", root.ptr);
    }
    git_buf_free(&root);
    git_libgit2_shutdown();
    return ret;
}
```

[repo相关函数](https://libgit2.org/libgit2/#HEAD/group/repository)

## sha转oid

>oid: (commit, tree, blob, tag)对象的唯一标识．

```c
int sha2oid() {
    const char *sha = "2c25bbcf03ef6bf51bbc7d8d7a443c4d079ca14d";
    git_oid oid = {0};
    git_oid_fromraw(&oid, sha);

    char shortsha[10] = {0};
    git_oid_tostr(shortsha, 9, &oid);
    printf("short sha: %s\n", shortsha);
}
```

## 获取最新commit

```c
int lookup_commit() {
    const char *repo_path = "../repo/libgit2";
    git_libgit2_init();
    //open repo
    git_repository *repo = NULL;
    int ret = git_repository_open(&repo, repo_path);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("clone error: %s\n", err->message);
        return ret;
    }

    //get last commit info
    git_commit *commit;
    git_oid oid_parent_commit;

    ret = git_reference_name_to_id(&oid_parent_commit, repo, "HEAD");

    if (ret != 0) {
        git_error *err = git_error_last();
        printf("name to id error: %s\n", err->message);
        return ret;
    }

    ret = git_commit_lookup(&commit, repo, &oid_parent_commit);

    git_oid *commit_id = git_commit_id(commit);
    git_signature *sig = git_commit_author(commit);
    char *commit_message = git_commit_message(commit);

    char id[41] = {0};
    git_oid_fmt(id, commit_id);

    printf("commit %s\nAuthor: %s <%s>\nDate:  %s\n\n", id, sig->name, sig->email, ctime(&sig->when.time));
    printf(commit_message);

    git_commit_free(commit);

    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## 提交历史

```c
/**
 * 打印提交详情
 * @param commit
 * @return
 */
int print_commit_info(git_commit *commit) {
    printf("\n");
    git_oid *commit_id = git_commit_id(commit);
    git_signature *sig = git_commit_author(commit);
    char *commit_message = git_commit_message(commit);

    char id[41] = {0};
    git_oid_fmt(id, commit_id);

    printf("commit %s\nAuthor: %s <%s>\nDate:  %s\n\n", id, sig->name, sig->email, ctime(&sig->when.time));
    printf(commit_message);
}
```
```c
/**
 * 提交历史
 * @return
 */
int commit_history() {
    const char *repo_path = "../repo/libgit2";
    git_libgit2_init();
    //open repo
    git_repository *repo = NULL;
    int ret = git_repository_open(&repo, repo_path);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("clone error: %s\n", err->message);
        return ret;
    }

    //get last commit info
    git_commit *commit;
    git_oid oid_parent_commit;

    ret = git_reference_name_to_id(&oid_parent_commit, repo, "HEAD");

    if (ret != 0) {
        git_error *err = git_error_last();
        printf("name to id error: %s\n", err->message);
        return ret;
    }

    ret = git_commit_lookup(&commit, repo, &oid_parent_commit);

    ret = print_commit_info(commit);

    //get commit history
    do {
        git_commit *nth_parent = NULL;
        ret = git_commit_parent(&nth_parent, commit, 0);
        if (ret != 0) {
            git_error *err = git_error_last();
            printf("error: %s\n", err->message);
            break;
        }
        print_commit_info(nth_parent);
        commit = nth_parent;
        git_commit_free(nth_parent);
    } while (ret == 0);

    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

[commit相关函数](https://libgit2.org/libgit2/#HEAD/group/commit)

