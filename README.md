# Backpack

Easily backup files to different storages and retrive them.

Supported storages:

- ftp
- move (move to local folder)
- AWS S3

# Usage

Usage: `backpack <command> [args] [flags]`

Flags:

| Flag    | Type | Description    | Required | Default |
| ------- | ---- | -------------- | -------- | ------- |
| version | bool | Print version. | false    | false   |

Global Flags (all commands has these):

| Flag    | Type | Description                                         | Required | Default |
| ------- | ---- | --------------------------------------------------- | -------- | ------- |
| debug   | bool | Enable debug mode. MAY PRINT SENSITIVE INFORMATION. | false    | false   |
| h, help | bool | Displays help                                       | false    | false   |

Commands:

- [backup](#backup)
- [restore](#restore)
- [test-connections](#test-connections)
- [test-format](#test-format)
- [version](#version)

## backup

Usage: `backpack backup [flags] --config <path to config>`

The backup command zips the folder specified as `data_path` in config file, encrypts it if `encryption` is set in config file, then runs the actions from config file. `data_path` isn't touched.

Flags:

| Flag       | Type             | Description                                                      | Required | Default    |
| ---------- | ---------------- | ---------------------------------------------------------------- | -------- | ---------- |
| c, config  | string           | Path to config file                                              | true     | no default |
| except     | array of strings | Doesn't backup to those actions. Can't be used with `--only`     | false    | no default |
| only       | array of strings | Does only backup to those actions. Can't be used with `--except` | false    | no default |
| force      | bool             | Backups even if data hasn't changed                              | false    | false      |
| no-encrypt | bool             | Doesn't encrypt data before uploading backup.                    | false    | false      |

## restore

Usage: `backpack restore [flags] --config <path to config>`

The restore command creates a new backup, downloads the file from an action selected in the interactive prompt and then decrypts and extrat data to `data_path`. What was at `data_path` before command is run will get removed.

Flags:

| Flag       | Type             | Description                                                                           | Required | Default    |
| ---------- | ---------------- | ------------------------------------------------------------------------------------- | -------- | ---------- |
| c, config  | string           | Path to config file                                                                   | true     | no default |
| except     | array of strings | Doesn't backup to those actions. Can't be used with `--only`                          | false    | no default |
| only       | array of strings | Does only backup to those actions. Can't be used with `--except`                      | false    | no default |
| action     | string           | Selects which action to download the data from. Avoids interactive prompt.            | false    | no default |
| file       | string           | Selects which file to download. Avoids interactive prompt.                            | false    | no default |
| force      | bool             | Backups even if data hasn't changed.                                                  | false    | false      |
| no-encrypt | bool             | Doesn't encrypt data before uploading backup and doesn't decrypt data after download. | false    | false      |
| no-backup  | bool             | Doesn't create backup before restoring                                                | false    | false      |

## test-connections

Usage: `backpack test-connections [flags] --config <path to config>`

The test-connections command validates that all actions are able to be connected to.

Flags:

| Flag      | Type             | Description                                                              | Required | Default    |
| --------- | ---------------- | ------------------------------------------------------------------------ | -------- | ---------- |
| c, config | string           | Path to config file                                                      | true     | no default |
| except    | array of strings | Doesn't try to connect to those actions. Can't be used with `--only`     | false    | no default |
| only      | array of strings | Does only try to connect to those actions. Can't be used with `--except` | false    | no default |

## test-format

Usage: `backpack test-format --config <path to config>`

The test-format command prints the formatted file name using the current time. This is so that you can validate that the format is what you expect.

Flags:

| Flag      | Type             | Description                                                              | Required | Default    |
| --------- | ---------------- | ------------------------------------------------------------------------ | -------- | ---------- |
| c, config | string           | Path to config file                                                      | true     | no default |

## version

Usage: `backpack version`

The version command prints the version.

# Config File

The config file is a yml file with all the information to be able to store and fetch files.

## Fields

| Key        | Type   | Description                                                                                                                                                                  | Required |
| ---------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| data_path  | string | Path to the folder that will be backed up or replaced when restoring                                                                                                         | true     |
| hash_path  | string | Path to a file containing the sha512 hash from the last backup. This is to prevent duplicate backups.                                                                        | true     |
| file_name_format  | string | Format of the file name used when backing up. Read section [File Name Format](#file-name-format)                                                                       | false     |
| cwd        | string | If defined then the program will move into that folder making relative path relative to cwd. (`--config` flag will still be relative to where the program was executed from) | false    |
| encryption | map    | Information about encryption data. Read section [Encryption](#encryption)                                                                                                    | false    |
| actions    | map    | List of different storages to backup to. Read section [Actions](#actions)                                                                                                    | true     |

### File Name Format

Format of the name to use when backing up. It has support for all datetime formats that golang offers aswell as some custom.
Default is `%Y-%m-%d_%H%M%S`. You can add more information if you want, like `prod_db_%Y-%m-%d_%H%M%S`.

#### Custom formats

| Code | Description | Golang Equivelent |
| ---- | ----------- | ----------------- |
| %a   | The abbreviated weekday name. | Mon |
| %A   | The full weekday name. | Monday |
| %d   | Day of the month (01 to 31). | 02 |
| %b   | The abbreviated month name. | Jan |
| %B   | The full month name. | January |
| %m   | Month of the year (01 to 12). | 01 |
| %Y   | Year with century. | 2006 |
| %y   | Year without a century. | 06 |
| %H   | Hour of the day (00 to 23). | 15 |
| %M   | Minute of the hour (00 to 59). | 04 |
| %S   | Seconds of the minute (00 to 60). | 05 |
| %Z   | Time zone name. | MST |

### Encryption

If encryption isn't set then backups won't be encrypted.

| Key  | Type | Description           | Required |
| ---- | ---- | --------------------- | -------- |
| type | enum | Allowed values: `aes` | true     |

Types:

- [aes](#aes)

#### aes

The strength of the encryption is determined by key size. 16 byte key will use AES-128, 24 byte key will use AES-192 and 32 byte key will use AES-256.

| Key | Type   | Description                                                           | Required |
| --- | ------ | --------------------------------------------------------------------- | -------- |
| key | string | Depending on prefix, the value will be parsed differently. See below. | true     |

Prefix:

- raw: Reads the rest as UTF-8 and uses that as a key. (Must be 32, 24 or 16 characters in length)
- hex: HEX encoded bytes. (Must be 64, 48 or 32 characters in length)
- base64: Base64 encoded bytes.
- file: Path to a file that will get read.

##### Example

```yaml
# rest of the config...
encryption:
  type: aes
  # Only one key can be used at a time
  key: raw:thisis32bitlongpassphraseimusing
  key: hex:74686973697333326269746c6f6e6770617373706872617365696d7573696e67
  key: base64:dGhpc2lzMzJiaXRsb25ncGFzc3BocmFzZWltdXNpbmc=
  key: file:test/key.txt
# rest of the config...
```

### Actions

Actions are named by there key and the config that contains depends on what type it has.

| Key  | Type | Description                            | Required |
| ---- | ---- | -------------------------------------- | -------- |
| type | enum | Allowed values: `move`, `ftp` and `s3` | true     |

Types:

- [move](#move)
- [ftp](#ftp)
- [s3](#s3)

#### move

| Key | Type   | Description                                                   | Required |
| --- | ------ | ------------------------------------------------------------- | -------- |
| dir | string | Path to a directory that the data will be stored and fetched. | true     |

##### Example

```yaml
# rest of the config...
actions:
  move_to_tmp:
    type: move
    dir: /tmp
  # rest of actions...
# rest of the config...
```

#### ftp

| Key      | Type   | Description                                                                   | Required | default    |
| -------- | ------ | ----------------------------------------------------------------------------- | -------- | ---------- |
| user     | string | User to log in as.                                                            | true     | no default |
| password | string | The password for the user.                                                    | false    | ""         |
| host     | string | URL or IP to the server.                                                      | true     | no default |
| port     | number | The port that the server uses.                                                | false    | 21         |
| dir      | string | Path to the directory on the server that the data will be stored and fetched. | false    | "/"        |

##### Example

```yaml
# rest of the config...
actions:
  ftp_anonymous:
    type: ftp
    user: anonymous
    host: 127.0.0.1
    dir: /uploads

  ftp_different_port:
    type: ftp
    user: user1
    password: Very_$ecure_passw0rd
    host: 127.0.0.1
    port: 603
  # rest of actions...
# rest of the config...
```

#### s3

| Key           | Type   | Description        | Required |
| ------------- | ------ | ------------------ | -------- |
| region        | string | AWS server region. | true     |
| bucket        | string | Name of bucked.    | true     |
| client_secret | string | AWS Client Secret. | true     |
| client_id     | string | AWS Client ID.     | true     |

##### Example

```yaml
# rest of the config...
actions:
  s3_my_bucket:
    type: s3
    region: eu-north-1
    bucket: my-bucket
    client_secret: ajdkwajd
    client_id: akwajdkawdk
  # rest of actions...
# rest of the config...
```

## Example

```yaml
data_path: database/data
prev_hash: ./prev_hash
cwd: /home/user1/mywebsite # remove this if you want cwd to be where you executed the program
file_name_format: 'prod_database_%Y-%m-%d_%H%M' # remove this if you want the default of '%Y-%m-%d_%H%M%S'

encryption: # Remove this if you don't want encryption
  type: aes
  key: raw:thisis32bitlongpassphraseimusing

actions:
  s3_my_bucket:
    type: s3
    region: eu-north-1
    bucket: my-bucket
    client_secret: ajdkwajd
    client_id: akwajdkawdk

  ftp_local:
    type: ftp
    user: user1
    password: Very_$ecure_passw0rd
    host: 127.0.0.1
    port: 603

  move_to_tmp:
    type: move
    dir: /tmp
```

# License

The MIT License (MIT)

Copyright © 2022 Marcus Nilsson <marcus.nilsson@genarp.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
