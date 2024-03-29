# bcl_ee
# The execution environment needed to set up the BCL demo in the CIC
# here are the steps to create it

# Using Github actions
We created a Github action which builds the execution environment twice and pushes it into quay.io
The build process is triggered manualy:

Navigate to
https://github.com/Red-Hat-EMEA-Portfolio-SSA/bcl_ee

Click on "Actions"
Click on "Build an ee, triggered manually" on the right hand side:
Drop down the dropdown "Run workflow" 
add a version number

and click "Run Workflow" 

Advantage of this porcess:
* way faster than my manual build process
* many steps combined into one!

Caution: It does NOT push it into the private automation hub in the BCL environment


# Using conventional build process
Set the version in a variable to allow cut and past in the following
Convention: 
   use a "n" behind your version number to emphasis it is has NO vsan modules included.
   use "vsan" behind your version number to emphasis it is has vsan modules included.

 ```
 export eev=15
 ```
Login to quay and RH registry to download things needed
 ```
 podman login quay.io
 ```
 ```
 podman login registry.redhat.io
 ```
Optional:
The build process is long lasting and has a long output. IF you want to review output later you might want to capture it. One option is via tmux
Start tmux:
 ```
 tmux
 ```
Start logging within tmux
 ```
 ctrl-b :
 pipe-pane -o 'cat >>~/tmux_output.#S:#I-#P'
 ```

Build a new execution environment 
 ```
 ansible-builder build -v 3 -t bcl-ov:${eev} -f execution-environment.yml --prune-images
 ```

# Version 14 and above > Including vSAN management 

For using the modules that manage vSAN with the community.vmware collection you nepodman push bcl-ov:4 quay.io/mschreie/bcl-ov:ed to include the vSAN Management python SDK. This SDK is provided and managed by VMware in a way that cannot just be included in the requirements.txt file. Here I explain how to include it in your execution environment to build version 14 of bcl_ee and above.

First thing you need to do is dowload the "vsan-sdk-python.zip" [from VMware site](https://developer.vmware.com/web/sdk/7.0%20U2/vsan-python) and place the file in the directory of this repo: /your-path/bcl_ee/vsan-sdk-python.zip

Once you have it there you need to execute ansible-builder with the schema version 3 file:

```
ansible-builder build -v 3 -t bcl-ov:${eev} -f ee-schema-v3.yml --prune-images
```

Then follow the next steps (pushing it everywhere needed)

# and push to quay.io
 ```
 podman push bcl-ov:${eev} quay.io/mschreie/bcl-ov:${eev}
 podman push bcl-ov:${eev} quay.io/mschreie/bcl-ov:latest
 ```

# and push to private automation hub
 ```
 podman login --tls-verify=false aap-privhub.bcl.redhat.hpecic.com
 Username: admin
```
```
 podman push --tls-verify=false bcl-ov:${eev} aap-privhub.bcl.redhat.hpecic.com/bcl-ov:${eev}
 podman push --tls-verify=false bcl-ov:${eev} aap-privhub.bcl.redhat.hpecic.com/bcl-ov:latest
 ```
