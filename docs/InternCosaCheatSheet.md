# Intern Cosa CheatSheet #

## Key Terms ##
## .bashrc ##
.bashrc is a shell script that Bash runs when a user logs in. It contains a bunch of configurations for the terminal session (command aliases, coloring, completion and shell history).

## podman ##
Podman is a open source container engine developed by RedHat.

### Add a bash alias / edit .bashrc ###

```
nano /home/YourUsernameHere/.bashrc
```

## Useful bash alias: ##

``` bash
cosa() {
   env | grep COREOS_ASSEMBLER
   local -r COREOS_ASSEMBLER_CONTAINER_LATEST="quay.io/coreos-assembler/coreos-assembler:latest"
   if [[ -z ${COREOS_ASSEMBLER_CONTAINER} ]] && $(podman image exists ${COREOS_ASSEMBLER_CONTAINER_LATEST}); then
       local -r cosa_build_date_str="$(podman inspect -f "{{.Created}}" ${COREOS_ASSEMBLER_CONTAINER_LATEST} | awk '{print $1}')"
       local -r cosa_build_date="$(date -d ${cosa_build_date_str} +%s)"
       if [[ $(date +%s) -ge $((cosa_build_date + 60*60*24*7)) ]] ; then
         echo -e "\e[0;33m----" >&2
         echo "The COSA container image is more that a week old and likely outdated." >&2
         echo "You should pull the latest version with:" >&2
         echo "podman pull ${COREOS_ASSEMBLER_CONTAINER_LATEST}" >&2
         echo -e "----\e[0m" >&2
         sleep 10
       fi
   fi
   set -x
   podman run --rm -ti --security-opt label=disable --privileged                                      \
              --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap 1001:1001:64536                            \
              -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                    \
              --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa                                           \
              ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}     \
              ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}    \
              ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS:+-v $COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS} \
              ${COREOS_ASSEMBLER_CONTAINER:-$COREOS_ASSEMBLER_CONTAINER_LATEST} "$@"
   rc=$?; set +x; return $rc
}
```
## Speeding up COSA edit-compile-debug cycle ##

`cosa` is a containerized collection of tools. When we make a change to one of these tools, we can run a new container build to test our changes; however, a better (and faster) option is to mount the changed files into `cosa`. 

We can edit our `~/.bashrc` cosa alias. In the example below, we will mount kola binaries to test changes to kola:

```
export COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS=/some/path/to/coreos-assembler/mantle/bin/:/usr/local/bin/:ro
```

Remember to substitute `/some/path/to/coreos-assembler/mantle/bin/` for the path to the kola binaries on your machine. With he `mantle/bin` folder mounted
every time you run `make` to compile kola, the changes will be picked up in your cosa container.

Tip: I personally don't like to edit my `~/.bashrc` file everytime. So I keep `cosa` bash scripts handy:

``` bash
$ pwd
~/my-build-directory
$ cat cosa
cosa() {
   env | grep COREOS_ASSEMBLER
   local -r COREOS_ASSEMBLER_CONTAINER_LATEST="quay.io/coreos-assembler/coreos-assembler:latest"
   if [[ -z ${COREOS_ASSEMBLER_CONTAINER} ]] && $(podman image exists ${COREOS_ASSEMBLER_CONTAINER_LATEST}); then
       local -r cosa_build_date_str="$(podman inspect -f "{{.Created}}" ${COREOS_ASSEMBLER_CONTAINER_LATEST} | awk '{print $1}')"
       local -r cosa_build_date="$(date -d ${cosa_build_date_str} +%s)"
       if [[ $(date +%s) -ge $((cosa_build_date + 60*60*24*7)) ]] ; then
         echo -e "\e[0;33m----" >&2
         echo "The COSA container image is more that a week old and likely outdated." >&2
         echo "You should pull the latest version with:" >&2
         echo "podman pull ${COREOS_ASSEMBLER_CONTAINER_LATEST}" >&2
         echo -e "----\e[0m" >&2
         sleep 10
       fi
   fi
   set -x
   podman run --rm -ti --security-opt label=disable --privileged                                    \
              --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap 1001:1001:64536                          \
              -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                  \
              --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa                                         \
              ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
              ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
              ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
              ${COREOS_ASSEMBLER_CONTAINER:-$COREOS_ASSEMBLER_CONTAINER_LATEST} "$@"
   rc=$?; set +x; return $rc
}

export COREOS_ASSEMBLER_GIT=~/assembler/coreos-assembler/
```

We can use different files to mount different common directories. In this one, we mount `coreos-assembler/src` into the COSA container.

## Building Fedora CoreOS using COSA Tutorial ##
https://coreos.github.io/coreos-assembler/building-fcos/

## Common Errors ##
```
Error: error creating container storage: the container name "cosa" is already in use by "694975cfc9765382a8977337a8186b2f854dab171170e80be6ea484d44622e96". You have to remove that container to be able to reuse that name.: that name is already in use.
```

Make sure to always keep the container at the latest version to prevent errors using:

```
podman pull quay.io/coreos-assembler/coreos-assembler:latestpodman pull quay.io/coreos-assembler/coreos-assembler:latest

```
## Solution ##

```
    podman ps -a

    podman stop nameofcontainerhere   
```

## Creating new Kola Test ##
Create a file in src/config/tests/kola. Create the file and make sure it has no extension. Use "chmod a=x testname" to add correct permissions.

You can also write a test that is compiled in Kola (ie. written in Go). In general opt for the first option (an external test) if possible, but if more complexity is needed, a test compiled in Kola is a good option.

## Running Kola Tests ##
https://coreos.github.io/coreos-assembler/kola/

First list all the kola tests using "cosa kola list". If you want to just get the name of your test you can use "cosa kola list | grep KeywordOfTest". Then you can run using "cosa kola run your.test.name.here".

## Seeing Results Of Your Test ##
Test results commonly held in a file inside  /non-exclusive-test-bucket-0/ so use ls to get the folder inside it and the console.txt with the results. 

E.g

```
ls tmp/kola/non-exclusive-test-bucket-0/

ls tmp/kola/non-exclusive-test-bucket-0/b525f4e2-93a9-47c5-your-folder-here/

cat tmp/kola/non-exclusive-test-bucket-0/b525f4e2-93a9-47c5-your-folder-here/console.txt

cat tmp/kola/non-exclusive-test-bucket-0/b525f4e2-93a9-47c5-your-folder-here/journal.txt
```

##### References ####
https://www.digitalocean.com/community/tutorials/bashrc-file-in-linux

https://coreos.github.io/coreos-assembler/kola/

https://unix.stackexchange.com/questions/129143/

what-is-the-purpose-of-bashrc-and-how-does-it-work

https://coreos.github.io/coreos-assembler/building-fcos/