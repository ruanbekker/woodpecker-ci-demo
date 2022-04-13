# woodpecker-ci-demo
woodpecker-ci-demo

## What is Woodpecker

Taken from their [documentation](https://woodpecker-ci.org/docs/intro): "Woodpecker is a simple CI engine with great extensibility. It runs your pipelines inside containers, so if you are already using them in your daily workflow, you'll love Woodpecker for sure."

You can read more about them [here](https://woodpecker-ci.org/docs/intro)

## About

This stack will boot the following:

- Gitea (Version Control)
- Woodpecker Server (Control Plane) 
- Woodpecker Agent (Responsible for running builds)

## Requirements

We will need the IP Address of your workstation (LAN), in order to enable the containers to communicate with each other (woodpecker-server and gitea).

This can be retrieved by running:

```bash
ifconfig $(netstat -rn | grep -E "^default|^0.0.0.0" | head -1 | awk '{print $NF}') | grep 'inet ' | awk '{print $2}' | grep -Eo '([0-9]*\.){3}[0-9]*'
```

In my case the output is:

```bash
192.168.0.182
```

Set that to the environment:

```
export IP_ADDRESS=$(ifconfig $(netstat -rn | grep -E "^default|^0.0.0.0" | head -1 | awk '{print $NF}') | grep 'inet ' | awk '{print $2}' | grep -Eo '([0-9]*\.){3}[0-9]*')

# or

export IP_ADDRESS=192.168.0.182 # replace with your ip address
```

## Gitea

Run the gitea container:

```bash
docker-compose up -d woodpecker-gitea
```

Access gitea on `http://git.${IP_ADDRESS}.nip.io:3000`, to display the url in your terminal:

```bash
echo http://git.${IP_ADDRESS}.nip.io:3000
```

Most of the defaults should be populated as we defined them in our environments section of our compose, at the bottom, provide the administrator details, then select "Install Gitea":

<img width="1088" alt="image" src="https://user-images.githubusercontent.com/567298/163272509-11e317ca-337f-4314-bcc3-82e18773181d.png">

Then you should be redirected to the main gitea menu:

<img width="1089" alt="image" src="https://user-images.githubusercontent.com/567298/163272782-90f2dcbf-1a68-4e88-a46f-25de97601349.png">

At this point in time we need to setup a OAuth application in order for our Woodpecker Server to authenticate with Gitea. Select the profile at the right top corner, then select settings:

<img width="414" alt="image" src="https://user-images.githubusercontent.com/567298/163273031-2ccdbe54-fadb-4ca0-bffe-099f8a0c6277.png">

Then select the "Applications" tab, then under "Manage OAuth2 Applications", provide the following:

- Application Name: `Woodpecker CI`
- Redirect URI: `http://ci.your-ip-address.nip.io:8000/authorize`

<img width="1082" alt="image" src="https://user-images.githubusercontent.com/567298/163273219-63e08459-5a5b-4e3e-addd-bdab8a264ac9.png">

Then click "Create Application" and you should get the following values:

- Client ID: `c587c627-cc4e-4fb1-a196-97736051090b`             # values has been deleted
- Client Secret: `u92Jvf-gxj_phcRnHWjwpmJv0NXoWthNGsXybds6CuE=` # values has been deleted

Then select "Save"

## Woodpecker

Time to configure the woodpecker server and agent, and all that we need to configure is the `.env` file so that we can make woodpecker aware of the gitea client and secret and as well as the woodpecker agent secret which is a shared secret between the woodpecker server and agent.

First let's create a agent secret:

```bash
$ openssl rand -base64 32
V5og1c7yAa5aZw7n7pjP86+jCkdRl3VakEy+EjC7vO8=
```

Then we will populate the gitea client and secret that we retrieved from gitea, and the agent secret that we received from the openssl command in the `.env` file:

```
WOODPECKER_AGENT_SECRET=V5og1c7yAa5aZw7n7pjP86+jCkdRl3VakEy+EjC7vO8=
WOODPECKER_GITEA_CLIENT=c587c627-cc4e-4fb1-a196-97736051090b
WOODPECKER_GITEA_SECRET=u92Jvf-gxj_phcRnHWjwpmJv0NXoWthNGsXybds6CuE=
```

Once that is in place, we can start the woodpecker server and agent containers:

```bash
docker-compose up -d
```

Now access woodpecker server on `http://ci.${IP_ADDRESS}.nip.io:8000`, to return the address in your terminal:

```bash
echo http://ci.${IP_ADDRESS}.nip.io:8000
```

On the initial page we will see the login screen:

<img width="1087" alt="image" src="https://user-images.githubusercontent.com/567298/163274634-abae7079-f20e-4ce8-a367-8140a52c9a0f.png">

Once we select login, we will be redirected to gitea where it will ask us if we authorize woodpecker ci (the application that we created) to access our gitea account:

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/567298/163274746-8867267b-a3c2-4463-aeaf-251d06c7b93d.png">

Select "Authorize Application", then we will be logged into woodpecker server:

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/567298/163274961-f8d445fb-5a39-4b24-a203-a6151e925d2c.png">

Head back to gitea, and from the top right hand corner, select "+" and select "New Repository":

<img width="410" alt="image" src="https://user-images.githubusercontent.com/567298/163275125-33d3751f-46a2-43aa-9d3c-33a033418528.png">

Provide a repository name and select "Initialize Repository":

<img width="1061" alt="image" src="https://user-images.githubusercontent.com/567298/163275314-1e1cce14-0e3d-4e5a-b52f-a433a820cd7d.png">

Then select "Create Repository":

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/567298/163275478-8bf1d173-89d3-4286-9b58-3cefc79806a3.png">

Head back to woodpecker server, then on the right hand side select "Add Repository":

<img width="1059" alt="image" src="https://user-images.githubusercontent.com/567298/163275621-5c1b1e2a-2f9e-4e51-81b2-61ea59e7d804.png">

Then select "Reload Repositories", and you should see your repository appear:

<img width="1058" alt="image" src="https://user-images.githubusercontent.com/567298/163275706-17b780a4-31e4-475a-96f8-6a5a0b9e03c5.png">

Then select "enable" and you should see a view with no builds:

<img width="1061" alt="image" src="https://user-images.githubusercontent.com/567298/163275779-4447b2bd-3551-45bb-90d4-9c0ea9bf5ed1.png">

Head back to gitea, select "new file", name the file `.woodpecker.yml` and provide this basic pipeline yaml:

```yaml
pipeline:
  first-job:
    image: busybox
    commands:
      - echo "first run"
```

And what we do, we specify that our build should run a container from a busybox image, and run the command echo first run. 

Commit the file to the master branch, then head back to woodpecker server, we will see that from the pipeline overview page, that our build succeeded:

<img width="1058" alt="image" src="https://user-images.githubusercontent.com/567298/163276435-0be62e76-8fd9-4d18-8224-5f45ed4294be.png">

When we select the build, we can see our build consisted of a clone step and our step which we called "first-job":

<img width="1058" alt="image" src="https://user-images.githubusercontent.com/567298/163276517-c1e219a5-6ec5-4137-a585-81736143de10.png">

## Example Pipelines

The following pipeline will run a mysql service container and our build step will use a container from the mysql image, to test the connectivity to the mysql service, it will retry until it establishes a connection and then exit:

```yaml
pipeline:
  build:
    image: mysql:5.7
    commands:
      - while ! mysqladmin ping -h mysql-server -u woodpecker -pwoodpecker --silent; do sleep 1; done
      
services:
  mysql-server:
    image: mysql:5.7
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: yes
      MYSQL_DATABASE: woodpecker
      MYSQL_USER: woodpecker
      MYSQL_PASSWORD: woodpecker
```

The following pipeline will run two steps in parallel (run-one and run-two) as they share the same group name, then step run-three will run after that, and we define the pipeline to only trigger when the branches includes master and any prefixes starts with releases/. The pipeline will not trigger on branches starting with test/1.0.0 and test/1.1.*:

```yaml
pipeline:
  run-one:
    image: busybox
    group: first
    commands:
      - echo "first run"
      
  run-two:
    image: busybox
    group: first
    commands:
      - echo "second run"

  run-three:
    image: ubuntu
    commands:
      - echo hi
when:
  branch:
    include: [ master, release/* ]
    exclude: [ test/1.0.0, test/1.1.* ]
```

For more information on the pipeline syntax, view:
- https://woodpecker-ci.org/docs/usage/pipeline-syntax
