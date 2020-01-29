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

## 显示commit中tree详情

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

    char cid[41] = {0};
    git_oid_fmt(cid, commit_id);

    printf("commit %s\nAuthor: %s <%s>\nDate:  %s\n\n", cid, sig->name, sig->email, ctime(&sig->when.time));
    printf(commit_message);
    printf("\n");

    git_tree *tree = NULL;
    int ret = git_commit_tree(&tree, commit);

    size_t entry_count = git_tree_entrycount(tree);
    for (int i = 0; i < entry_count; ++i) {
        git_tree_entry *entry = git_tree_entry_byindex(tree, i);
        const char *entry_name = git_tree_entry_name(entry);
        git_object_t entry_type_enum = git_tree_entry_type(entry);
        git_oid *entry_id = git_tree_entry_id(entry);
        git_filemode_t entry_filemode = git_tree_entry_filemode_raw(entry);

        char eid[41] = {0};
        git_oid_fmt(eid, entry_id);

        const char *entry_type_str = git_object_type2string(entry_type_enum);

        //output: filemoode type sha1 filename
        printf("%07o\t%s\t%s\t%s\n", (int)entry_filemode, entry_type_str, eid, entry_name);
    }

    printf("\n--------------------------------------------------------------------\n");
}
```

输出格式如下：
```
0100644	blob	fd8430bc864cfcd5f10e5590f8a447e01b942bfe	.HEADER
0100644	blob	34c5e9234ec18c69a16828dbc9633a95f0253fe9	.editorconfig
0100644	blob	176a458f94e0ea5272ce67c36bf30b6be9caf623	.gitattributes
0040000	tree	e8bfe5af39579a7e4898bb23f3a76a72c368cee6	.github
0100644	blob	dec3dca06c8fdc1dd7d426bb148b7f99355eaaed	.gitignore
```

## 查看引用

>```git show-ref```命令查看所有引用

```c
/**
 * 打印引用详情
 * @param repo 
 * @param ref_name 
 */
void print_ref_info(const git_repository *repo, const char *ref_name) {
    git_reference *ref = NULL;
    git_reference_lookup(&ref, repo, ref_name);
    git_reference_t ref_type = git_reference_type(ref);
    //ref_oid即commit id
    git_oid *ref_oid = git_reference_target(ref);

    char rid[41] = {0};
    git_oid_fmt(rid, ref_oid);

    printf("ref name: %s    type: %d,    id: %s\n", ref_name, ref_type, rid);
}
```

```c
/**
 * 引用列表
 * @return
 */
int lookup_refs() {
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

    //list refs
    git_strarray refs = {0};
    ret = git_reference_list(&refs, repo);

    for (int i = 0; i < refs.count; ++i) {
        print_ref_info(repo, refs.strings[i]);
    }

    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## tag列表

```c
/**
 * 打印tag详情
 */
void print_tag_info(git_tag *tag) {
    git_object_t tag_type = git_tag_target_type(tag);
    const char *tag_name = git_tag_name(tag);
    const git_signature *tagger = git_tag_tagger(tag);
    const char *message = git_tag_message(tag);

    printf("Name: %s\nType: %d\nTagger: %s\neMail: %s\nMessage: %s\n\n", tag_name, tag_type, tagger->name,
           tagger->email, message);
}
```

```c
/**
 * 遍历tag，只输出附注标签
 * @param name
 * @param tag_id
 * @param payload
 * @return
 */
int each_tag(const char *name, git_oid *tag_id, void *payload) {
    git_repository *repo = (git_repository *) payload;
    git_object *obj = NULL;
    int ret = git_revparse_single(&obj, repo, name);

    if (ret == 0) {
        git_object_t type = git_object_type(obj);
        const char *type_str = git_object_type2string(type);
        switch (type) {
            //附注标签
            case GIT_OBJECT_TAG:
                print_tag_info((git_tag *) obj);
                break;
            default:
                printf("%s[%s]\n", name, type_str);
                break;
        }
    } else {
        git_error *err = git_error_last();
        printf("error: %s, tag: %s\n", err->message, name);
    }
    return 0;
}
```

```c
/**
 * tag列表
 * @return
 */
int list_tags() {
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

    //list tags
    git_strarray tags = {0};

    git_tag_list(&tags, repo);
    git_tag_foreach(repo, each_tag, repo);

    for (int i = 0; i < tags.count; ++i) {
//        printf("%s\n", tags.strings[i]);
    }

    git_repository_free(repo);
    git_libgit2_shutdown();
    return ret;
}
```

## index文件读取

> 命令```git ls-files --stage```可以读取index文件

```c
/**
 * 打印index详情
 * @param index
 */
void print_index_info(git_index *index) {
    //file .git/index命令查看版本和entry数
    unsigned int index_version = git_index_version(index);
    unsigned int entry_count = git_index_entrycount(index);
    printf("index: Git index, version %d, %d entries\n", index_version, entry_count);


    //查看index中所有entry　git ls-files --stage
    //命令输出：<mode> <object> <stage> <file>

    git_index_iterator *iter = NULL;
    int ret = git_index_iterator_new(&iter, index);

    git_index_entry *entry = NULL;

    int iter_ret = 0;
    do {
        iter_ret = git_index_iterator_next(&entry, iter);
        if (iter_ret == GIT_ITEROVER) {
            break;
        }

        char eid[41] = {0};

        git_oid_fmt(eid, &entry->id);

        int stage_number = git_index_entry_stage(entry);

        printf("%06o  %s  %d  %s\n", entry->mode, eid, stage_number, entry->path);
    } while (iter_ret == 0);

    git_index_iterator_free(iter);
}
```

```c
/**
 * 打开index
 * @return
 */
int open_index() {
    const char *index_path = "../repo/libgit2/.git/index";
    git_libgit2_init();

    git_index *index = NULL;
    int ret = git_index_open(&index, index_path);

    if (ret == 0) {
        print_index_info(index);
        git_index_free(index);
    }

    git_libgit2_shutdown();
}
```

## 状态

>git status查看当前仓库状态

```c
/**
 * 状态枚举转换字符串
 * @param status_flags 
 * @return 
 */
char *status_2_str(unsigned int status_flags) {
    if (status_flags == GIT_STATUS_CURRENT) {
        return "CURRENT";
    } else if (status_flags == GIT_STATUS_INDEX_NEW) {
        return "INDEX_NEW";
    } else if (status_flags == GIT_STATUS_INDEX_MODIFIED) {
        return "INDEX_MODIFIED";
    } else if (status_flags == GIT_STATUS_INDEX_DELETED) {
        return "INDEX_DELETED";
    } else if (status_flags == GIT_STATUS_INDEX_RENAMED) {
        return "INDEX_RENAMED";
    } else if (status_flags == GIT_STATUS_INDEX_TYPECHANGE) {
        return "INDEX_TYPECHANGE";
    } else if (status_flags == GIT_STATUS_WT_NEW) {
        return "WT_NEW";
    } else if (status_flags == GIT_STATUS_WT_MODIFIED) {
        return "WT_MODIFIED";
    } else if (status_flags == GIT_STATUS_WT_DELETED) {
        return "WT_DELETED";
    } else if (status_flags == GIT_STATUS_WT_TYPECHANGE) {
        return "WT_TYPECHANGE";
    } else if (status_flags == GIT_STATUS_WT_RENAMED) {
        return "WT_RENAMED";
    } else if (status_flags == GIT_STATUS_WT_UNREADABLE) {
        return "WT_UNREADABLE";
    } else if (status_flags == GIT_STATUS_IGNORED) {
        return "IGNORED";
    } else if (status_flags == GIT_STATUS_CONFLICTED) {
        return "CONFLICTED";
    } else {
        return "UNKNOWN";
    }
}
```

```c
int git_status_callback(const char *path, unsigned int status_flags, void *payload) {
    printf("%s     [%s]\n", path, status_2_str(status_flags));
    return 0;
}

int show_status() {
    git_libgit2_init();
    const char *repo_root_path = "../../repo/libgit2";
    git_repository *repo;
    int ret = git_repository_open(&repo, repo_root_path);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("open repo error: %s\n", err->message);
        return ret;
    }

    ret = git_status_foreach(repo, git_status_callback, NULL);

    git_libgit2_shutdown();

    return ret;
}
```

## 差异

```c
int git_diff_file_callback(const git_diff_delta *delta, float progress, void *payload) {
    printf("diff %s %s\n", delta->old_file.path, delta->new_file.path);
    return 0;
}

int git_diff_binary_callback(const git_diff_delta *delta, const git_diff_binary *binary, void *payload) {
    printf("binary\n");
    return 0;
}

int git_diff_hunk_callback(const git_diff_delta *delta, const git_diff_hunk *hunk, void *payload) {
    if (hunk) {
        printf("%s\n", hunk->header);
    }
    return 0;
}

int git_diff_line_callback(const git_diff_delta *delta, const git_diff_hunk *hunk, const git_diff_line *line,
                           void *payload) {
    if (line) {
        printf("%s", line->content);
    }
    return 0;
}
```

```c
/**
 * 
 * @return 
 */
int diff() {
    git_libgit2_init();
    const char *repo_root_path = "../../repo/libgit2";
    git_repository *repo;
    int ret = git_repository_open(&repo, repo_root_path);
    if (ret != 0) {
        git_error *err = git_error_last();
        printf("open repo error: %s\n", err->message);
        return ret;
    }

    git_diff *diff;

    ret = git_diff_index_to_workdir(&diff, repo, NULL, NULL);

    git_diff_stats *stats;

    ret = git_diff_get_stats(&stats, diff);

    git_buf buf = {0};

    ret = git_diff_stats_to_buf(&buf, stats, GIT_DIFF_STATS_FULL, 10);

    printf(buf.ptr);

    printf("\n");

    git_buf_free(&buf);

    ret = git_diff_foreach(diff, git_diff_file_callback, git_diff_binary_callback, git_diff_hunk_callback,
                           git_diff_line_callback, NULL);

    git_libgit2_shutdown();

    return ret;
}
```

## 配置

```c

int git_config_foreach_callback(const git_config_entry *entry, void *payload) {
    printf("%s=%s\n", entry->name, entry->value);
}

int show_config(const char *config_name, git_config *config) {
    printf("=========================%s config=========================\n", config_name);
    return git_config_foreach(config, git_config_foreach_callback, NULL);
}

int show_config_by_path(const char *config_name, const char *config_path) {
    git_config *config;
    int ret = git_config_open_ondisk(&config, config_path);
    if (ret == 0) {
        ret = show_config(config_name, config);
        git_config_free(config);
    }
    return ret;
}

/**
 * 查看配置
 * @return
 */
int find_config() {
    git_buf global_config_path = {0};
    git_buf system_config_path = {0};

    git_libgit2_init();
    int ret = git_config_find_global(&global_config_path);
    if (ret == 0) {
        printf("global config file path: %s\n", global_config_path.ptr);
        show_config_by_path("global", global_config_path.ptr);
    } else {
        printf("can't found global config file,");
        git_error *error = git_error_last();
        printf("%s\n", error->message);
    }

    ret = git_config_find_system(&system_config_path);
    if (ret == 0) {
        printf("system config file path: %s\n", system_config_path.ptr);
        show_config_by_path("system", system_config_path.ptr);
    } else {
        printf("can't found system config file,");
        git_error *error = git_error_last();
        printf("%s\n", error->message);
    }

    const char *repo_root_path = "../../repo/libgit2";
    git_repository *repo;
    ret = git_repository_open(&repo, repo_root_path);
    if (ret == 0) {
        git_config *config;
        ret = git_repository_config(&config, repo);
        show_config("repo", config);
    } else {
        printf("can't found repo config file,");
        git_error *error = git_error_last();
        printf("%s\n", error->message);
    }

    git_libgit2_shutdown();
    return ret;
}
```
