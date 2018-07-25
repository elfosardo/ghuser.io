> Work in progress - [#55](https://github.com/AurelienLourot/ghuser.io/issues/55)

# Database

It is here to cache GitHub's data. During the prototyping phrase, it's just a set of
[JSON files](data/) that we update once per day.

## Updating the database

Several scripts form a pipeline for updating the database. Here is the data flow:

```
[ ./addUser.js myUser ]   [ ./rmUser.js myUser ]
                 │             │
                 v             v
              ┌───────────────────┐
              │ users/myuser.json │<───────────┐
              └────────────────┬──┘ │─┐        │
                └──────────────│────┘ │        │                    ╔════════╗
                  └────┬───────│──────┘        │                    ║ GitHub ║
                       │       │               │                    ╚════╤═══╝
                       │       v               │                         │
                       │   [ ./fetchUserDetailsAndContribs.js myUser ]<──┤
                       │                                                 │
                       ├──────────────────────────>[ ./fetchOrgs.js ]<───┤
                       │                                    │            │
                       │                                    v            │
                       │                                ┌───────────┐    │
                       │                                │ orgs.json │    │
                       │                                └─────┬─────┘    │
                       │                                      │          │
                       ├──>[ ./fetchRepos.js ]<──────────────────────────┘
                       │             │                        │
                       │             v                        │
                       │  ┌───────────────────────────┐       │
                       │  │ repos/myOwner/myRepo.json │─┐     │
                       │  └───────────────────────────┘ │─┐   │
                       │    └───────────────────────────┘ │   │
                       │      └────┬──────────────────────┘   │
                       │           │                          │
                       │           │      ┌───────────────────┘
                       │           │      │
                       v           v      v
                   [ ./calculateContribs.js ]
                                 │
                                 v
                      ┌──────────────────────┐
                      │ contribs/myuser.json │─┐
                      └──────────────────────┘ │─┐
                        └──────────────────────┘ │
                          └──────────────────────┘
```

For running the entire data pipeline at once, do

```bash
$ npm install
$ export GITHUB_CLIENT_ID=0123456789abcdef0123
$ export GITHUB_CLIENT_SECRET=0123456789abcdef0123456789abcdef01234567
$ export GITHUB_USERNAME=AurelienLourot
$ export GITHUB_PASSWORD=********
$ ./fetchAndCalculateAll.sh
...
data/
  users/
    26 users
    largest: richardlitt.json (39 KB)
    total: 123 KB
  contribs/
    largest: richardlitt.json (287 KB)
    total: 766 KB
  repos/
    1733 repos
    largest: OpenGenus/cosmos.json (14 KB)
    total: 3832 KB
  orgs.json: 107 KB
  total: 4828 KB

=> 186 KB/user

real    10m35.411s
user    2m34.128s
sys     0m18.912s
```

> **NOTE**: API keys can be created [here](https://github.com/settings/developers).

## Serving one more user

```bash
$ ./addUser.js newGreatUser
$ ./fetchAndCalculateAll.sh
```

## Deleting a profile

```bash
$ ./rmUser.js formerUser "you asked us to remove your profile in https://github.com/AurelienLourot/ghuser.io/issues/666"
$ ./fetchAndCalculateAll.sh # will also perform some garbage collection
```