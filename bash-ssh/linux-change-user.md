# Linux Permission Commands

## 1. `chmod` — Change File Permissions

`chmod` changes who can read, write, or execute a file or directory.

### Permission Types

| Symbol | Meaning | Number |
|---|---|:---:|
| `r` | read | 4 |
| `w` | write | 2 | 
| `x` | execute | 1 |

### User Types

| Symbol | Meaning |
|---|---|
| `u` | user / owner |
| `g` | group |
| `o` | others |
| `a` | all |

### Examples

```bash
chmod 755 script.sh
````

Meaning:

```text
owner: read + write + execute
group: read + execute
others: read + execute
```

```bash
chmod +x script.sh
```

Adds execute permission.

---

## 2. `chown` — Change File Owner

`chown` changes the owner of a file or directory.

### Example

```bash
sudo chown ubuntu file.txt
```

Changes the owner of `file.txt` to `ubuntu`.

```bash
sudo chown ubuntu:ubuntu file.txt
```

Changes both owner and group to `ubuntu`.

```bash
sudo chown -R ubuntu:ubuntu /data
```

Changes owner and group recursively for `/data`.

---

## 3. `chgrp` — Change File Group

`chgrp` changes only the group of a file or directory.

### Example

```bash
sudo chgrp developers file.txt
```

Changes the group of `file.txt` to `developers`.

```bash
sudo chgrp -R developers /project
```

Changes the group recursively for `/project`.

---

## Quick Summary

|Command|Purpose|
|---|---|
|`chmod`|change permissions|
|`chown`|change owner|
|`chgrp`|change group|

---

## Common Example

```bash
sudo chown -R ubuntu:ubuntu /data
chmod 755 /data
```

Meaning:

```text
/data owner becomes ubuntu
/data group becomes ubuntu
owner can read/write/execute
group and others can read/execute
```

## usermod
- change current logged in user to specific group

```sh
sudo usermod -aG docker $USER
```

- `-a`: append to group docker
- `-G docker`: have $USER under `docker` group

## Groups

### `groups`
- list every group

### `newgrp` 
- create a new group