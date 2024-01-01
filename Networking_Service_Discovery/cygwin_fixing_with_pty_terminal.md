# How to fix Cygwin Terminal with WinPty

- if we are using the `cygwin terminal` and want to perform an `kubectl exec -it ` command then `it will be be a pain to have to TTY Terminal`

- here the `-it flag` will provide the `interactive teletype terminal` `connection` to the `POD`

- when we execute the command as below  then we can see the output as below

    
    ```bash
        kubectl exec -it webapp-7f58455867-k2sbd -- sh
        # accessing the POD terminal using the kubectl exec command in here
        # if we are in cygwin we will be getting an message as below
        Unable to use a TTY- input is not a terminal or right kind of file
        # here we can see un interactive terminal where we can post the command but can't go back to the preveious command and so on
        # this command line is not properly not laid out    
    ```

- while use it in `Production kubernetes system to do the troubleshooting` thats the `only purpose of using the kubectl exec command` for the `emergency situation` for the `low level debugging`

- but here we want to exec to the `WebApp Container POD` and try to access the `MYSQL DB POD Service command line` using the `kubectl exec -it <WebAPP POD> -- <command>` hence we need to install a `interactive terminal over here` 

- there is a `simple elegent solution avialable for the same`

  - we can use a `terminal solution` called `winpty` in order to `avoud this issue with the Teletype Terminal`
  
  - we can goto `github` &rarr; `https://github.com/rprichard/winpty` &rarr; `goto the release` &rarr; `install the cygwin-64 bit tar.gz file` as below

    
    ```bash
        curl -L https://github.com/rprichard/winpty/releases/download/0.4.3/winpty-0.4.3-cygwin-2.8.0-x64.tar.gz > winpty.tar.gz
        # here we are using the curl command with -L which stands for the redirect as github release url will redirrect to official tar.gz zip page
        # using the redirection we are uinstalling the winpty.tar.gz zip file 

        # then we can extract the winpty.tar.gz as below
        tar xvf winpty.tar.gz
        # extracting the zupfile over here

        # then we need to go to the winpty-0.4.3-cygwin-2.8.0-x64 folder as it will be created on extraction 
        cd winpty-0.4.3-cygwin-2.8.0-x64/bin
        # going to the bin folder of the winpty-folder which got extracted from the zip

        #now we need to add that to the cygwin PATH which can be done easily as below 
        cp * /usr/local/bin
        # add all the exe that reside in the bin folder to the /usr/local/bin which is the cygwin PATH

        #now when we use the winpty command we can see the output as below
        winpty
        # below will be output in that case out here
        Usage: winpty [option] [--] program [args]

        Options:
        -h , --help  show this help message
        --mouse      Enable terminal mouse input
        --showkey    Dump STDIN escape sequences
        --version    Show the winpty version number
    
    
    ```

- in order to access the `interactive teletype terminal` while using the `kubectl exec -it <POD name> -- <command>` we can do as below

- we can use it as below

    ```bash
        winpty kubectl exec -it webapp-7f58455867-k2sbd -- sh
        # then we can see the response as below with the proper terminal
        / $ ls # we can see here the terminal been comin  up now in this case
        bin    dev    etc    home   lib    media  mnt    proc   root   run    sbin   srv    sys    tmp    usr    var

    
    
    
    
    ```