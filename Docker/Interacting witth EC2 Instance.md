## Accessing EC2 Image

In /Users/christophercolclough

Run `sso`

This will open an authorization request from AWS in the default web browser and start a secure tunnel to the EC2 instance

run `ssmI`

This shows all the containers running on EC2 and you're using AWS SSM, which is basically an alternative to SSH.

Select the container you want to be inside of.  We'll pick "Admin"

Select `connect (ssm)`

Now a session has been created and the terminal will show the SessionId.  The working directory is also now set to /home/aws_username.  In my case it's "/home/colcloughc"


Run this command to create a new container
`docker run -it -e SOPS_KMS_ARN=arn:aws:kms:us-east-1:165131651646165156:key/1564651ghfdhrrt6514651651-31265-5fgdg5-aad6-514611124874 -v /Users/christophercolclough/Projects/ilr:/ilr -v ~/.gitconfig:/root/.gitconfig --entrypoint=/bin/bash --name chris-es 0160165132016565.dkr.ecr.us-east-1.amazonaws.com/library/terraform-container:latest`
### Explanation
`docker run` runs a container from a specified image

`-it` says to create an interactive terminal session

`-e SOPS_KMS_ARN=arn:aws...` sets the environment variable SOPS_KMS_ARN and points it to the AWS KMS key for SOPS (Secrets Operations) to enable decryption of configuration files.

`-v /Users/christophercolclough/Projects/ilr:/ilr` creates a Volume mount. The colon is the delimiter between the left and right sides
The Left side: your local path on macOS
Right side: inside the container

This makes your local ilr project accessible inside the container.

`-v ~/.gitconfig:/root/.gitconfig` creates a second volume mount, mapping your git configuration:

Left: your local Git config file
~/.gitconfig

Right: root user’s Git config inside the container
/root/.gitconfig

This lets Git commands inside the container use your host machine's git identity and settings.

`--entrypoint=/bin/bash`
Overrides the default entrypoint of the container image.
Instead of running Terraform or whatever the image normally starts, Docker will start a bash shell.
This is what makes your container drop you into a terminal session


`--name chris-es`
Assigns a custom container name

`165132561126.dkr.ecr.us-east-1.amazonaws.com/library/terraform-container:latest`

This is the AWS ECR image that is being pulled and run.


<br>

## Interacting with Docker image inside EC2 VM

`docker ps -a` will return the list of all containers with their ids, names, status, etc.

`docker exec -it chris-es bash` execs into the named container (chris-es) with an interactive bash terminal

