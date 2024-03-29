# Hands on with Docker - Build and manage your own images

__Disclaimer__: Parts of the session below were inspired and indeed reuse examples from the Cyverse documentation (Release 0.2.0; March 05, 2020; <a href="https://learning.cyverse.org/_/downloads/foss-2020/en/latest/pdf/" title="Cyverse documentation pdf" target="_blank" >pdf</a>).

The following session assumes that you have Docker running on your computer (we tested with Docker version 18.09.7, build 2d0083d). 
If you're running on a Linux host it further assumes that you have it set up in such a way that you don't have to prepend `sudo` to the `docker` command each time. There is a number of ways you can do that, e.g. by creating a docker group and adding your user to it or by running the Docker daemon as a non-root user (find some more info/instructions <a href="https://docs.docker.com/engine/install/linux-postinstall/" title="Post-installation steps for Linux (last accessed 24.04.2020)" target="_blank">here</a>). If for some reason you don't want to or can do any of these things, all of the below should also work if you simply prepend `sudo` to the `docker` command.

# Build your own image

In the previous sesssion you have been shown how to run existing, pre-build Docker containers. Now it's time to make your own Docker image running only a particular piece of software of your choosing.

As an example I have chosen `BLAST`, which is used to find regions of similarity between biological sequences, DNA- and/or Protein. The BLAST algorithm (Basic Local Alignment Search Tool) and it's derivatives is among the most used tools in DNA-sequence based research. There is an online version hosted by <a href="https://blast.ncbi.nlm.nih.gov/Blast.cgi" title="BLAST online" target="_blank" >NCBI</a>, but there are many situations where it's handy to have it running locally. 

## Interactive Build

Since you've already used it in the previous session we will be starting from a base Ubuntu image. Let's start a container interactively.

```bash
(host)-$ docker run -it -h my_manual_blast --name ${USER}s_manual_blast ubuntu:18.04
```
Note the additional flags I am using - these are optional:

* `-h` - Gives a defined name to the Docker host - just so it looks the same for all of us
* `--name` - Name the container

Your prompt should look like this now:
```bash
root@my_manual_blast:/#
```

Now, let's see if Ubuntu already comes with `BLAST` pre-installed, specifically a programm called `blastn`, which is used for comparing DNA sequences.
```bash
root@my_manual_blast:/$ blastn

bash: blastn: command not found
```

No. But Ubuntu has an online registry for software and it happens to contain the standard suite of BLAST programs. To set this up we need to update this system first. Ubuntu has a free-software user interface called `apt` (Advanced Package Tool) to manage installation and removal of software between your system and the registry.


```bash
root@my_manual_blast:/$ apt-get update
```

Note: If you're unsure if the program you want to install is in the registry and what the package is called (it's important to know exactly, because the command line is case sensitive and will not recognize your request if you have misstyped), `apt` has a function for searching the registry.
```bash
(host)-$ apt-cache search blast
```
This is not a great example, because you get lots of packages back. But somewhere in there you'll find `ncbi-blast+`. This is what you want.

Then we can install the desired program as follows using `apt-get`. You will be notified that the installation will require some additional diskspace and need to agree with typing `y`.
```bash
root@my_manual_blast:/$ apt-get install ncbi-blast+
```

If we try again now, it seems to work. 
```bash
root@my_manual_blast:/$ blastn -h
USAGE
  blastn [-h] [-help] [-import_search_strategy filename]
    [-export_search_strategy filename] [-task task_name] [-db database_name]
    [-dbsize num_letters] [-gilist filename] [-seqidlist filename]
    [-negative_gilist filename] [-entrez_query entrez_query]
    [-db_soft_mask filtering_algorithm] [-db_hard_mask filtering_algorithm]
    [-subject subject_input_file] [-subject_loc range] [-query input_file]
    [-out output_file] [-evalue evalue] [-word_size int_value]
    [-gapopen open_penalty] [-gapextend extend_penalty]
    [-perc_identity float_value] [-qcov_hsp_perc float_value]
    [-max_hsps int_value] [-xdrop_ungap float_value] [-xdrop_gap float_value]
    [-xdrop_gap_final float_value] [-searchsp int_value]
    [-sum_stats bool_value] [-penalty penalty] [-reward reward] [-no_greedy]
    [-min_raw_gapped_score int_value] [-template_type type]
    [-template_length int_value] [-dust DUST_options]
    [-filtering_db filtering_database]
    [-window_masker_taxid window_masker_taxid]
    [-window_masker_db window_masker_db] [-soft_masking soft_masking]
    [-ungapped] [-culling_limit int_value] [-best_hit_overhang float_value]
    [-best_hit_score_edge float_value] [-window_size int_value]
    [-off_diagonal_range int_value] [-use_index boolean] [-index_name string]
    [-lcase_masking] [-query_loc range] [-strand strand] [-parse_deflines]
    [-outfmt format] [-show_gis] [-num_descriptions int_value]
    [-num_alignments int_value] [-line_length line_length] [-html]
    [-max_target_seqs num_sequences] [-num_threads int_value] [-remote]
    [-version]

DESCRIPTION
   Nucleotide-Nucleotide BLAST 2.6.0+

Use '-help' to print detailed descriptions of command line arguments

```
Note, that I call the software and add a `-h` to the call. This is a very common, so-called ___flag___ in command line software that usually gives you some kind of help about the program. In this case it shows all the options the `blastn` program has. In this case there is even a more extensive help you can get by typing `blastn -help`.

Great! Now you have `blastn` running in a container. But how to make this permanent?

Let's exit the container and see what we can do. The following will bring your original prompt back.

```bash
root@my_manual_blast:/$ exit
```

Now, if you type `docker container ls -a` (in older docker versions this is the same as `docker ps -a`) you will see the list of all containers that you ran so far (the ones that are running as well as those which are already exited), including `user*s_manual_blast`, which you have just exited.

We can convert this container, including the changes you made to the base Ubuntu image to a new image - I will call it `user*s_manual_blast_image`. Docker has a subroutine for that, called `commit`. You need to also provide a commit message via the `-m` flag. This is usually short information about how you changed the image, so when you look at it later you will be able to remember what the changes were. 
```bash
(host)-$ docker commit -m "ubuntu + blast" ${USER}s_manual_blast ${USER}s_manual_blast_image
```

Great! Now the image should show up if you type `docker image ls`.

You can use it like we've done before. The `--rm` just tells Docker to remove the container once you're done. This is handy, otherwise you will accumulate excited containers very fast. Below I dropped the `--name` flag, because it's optional and at this stage I don't care which name the system gives to the container while it's running.
```bash
(host)-$ docker run -it --rm -h my_manual_blast ${USER}s_manual_blast_image

root@my_manual_blast:/$ blastn -h
USAGE
   .
   .
   .
root@manual_blast_image:/$ exit
```

You can use the image also like an executable, rather than interactively, try:
```bash
(host)-$ docker run --rm ${USER}s_manual_blast_image blastn -h
```

## Automatic Build

Docker can build images automatically, reading instructions from a text file, the so-called `Dockerfile`. This is simply a text document that contains all the commands you would normally execute manually in order to build your Docker image. 

A full reference to all features and extensions is provided at Docker's official documentation <a href="https://docs.docker.com/engine/reference/builder/" title="Dockerfile reference" target="_blank">here</a>.

Let's try to create your first `Dockerfile` and design it to build an image as the one we did manually above.

To keep things tidy, let's first make and move to a new directory.
```bash
(host)-$ mkdir automatic-blast && cd automatic-blast
```

Using your favorite text editor, create a file called `Dockerfile` and copy/paste the following text into it.
```
FROM ubuntu:18.04

RUN apt-get update

RUN apt-get install ncbi-blast+
```

Note that these are the exact same commands that we just ran interactively. We just prepend specific directives that will be interpreted by Docker.

* `FROM` - tells Docker to start building our image onto a certain base image. 
* `RUN` - The commands in each of these lines are actually executed during the process of building the image.

Let's try to build the image as instructed in the Dockerfile. Docker has a command for that. 
```bash
(host)-$ docker build -t ${USER}s_automatic_blast_image .
```
Note the flag `-t` which I use to name the image `automatic_blast_image`. The `.` is mandatory and just tells it to look for a file called `Dockerfile` (per default) in your current working directory. This behavior can be changed, but you can try to figure that one out for yourself if you want.

You will see the same information as before during system update, but then unfortunately we get an error. Remember you've been prompted to agree that extra diskspace is used before? Docker does not allow user interaction during build. Let's make a small change to the Dockerfile to fix that - just add `-y`, which is an option of the install command and just means 'don't prompt - yes to all'.

```
FROM ubuntu:18.04

RUN apt-get update

RUN apt-get install -y ncbi-blast+
```

Try again.

```bash
(host)-$ docker build -t ${USER}s_automatic_blast_image .
```

Looks good! Take a second to inspect the output Docker created and note that during the second build attempt Docker has not redone the update, but rather continued from from the first line in the Dockerfile that caused the error. 

If you type `docker image ls` now, the image should exist. We can try it out, like so:
```bash
(host)-$ docker run --rm ${USER}s_automatic_blast_image blastn -h
```

# Backup and share your container

## Create a local backup of your image

Docker allows you to create local backup of your custom image, that you can store away safely somewhere and/or share with your mates. Let's do that for the last image we've built.
```bash
(host)-$ docker save ${USER}s_automatic_blast_image > ${USER}s_automatic_blast_image.tar
```
You can restore the image any time from the archive that has been created. Let's live dangerously and remove the image - pretend it was an accident.
```bash
(host)-$ docker image rm ${USER}s_automatic_blast_image
```

Check `docker image ls` - Ups - it's gone. But, we can reload it from the archive.
```bash
(host)-$ docker load -i ${USER}s_automatic_blast_image.tar
```

## Share your image with the world - Dockerhub

Docker runs an online repository where users can deposit and host their images: <a href="https://hub.docker.com/" title="Dockerhub" target="_blank">Dockerhub</a>. An extensive documentation of what Dockerhub can do, far beyond what we can cover in today's introduction can be found in Docker's official Dockerhub documentation <a href="https://docs.docker.com/docker-hub/" title="Dockerhub documentation" target="_blank">here</a>.

In order to use it you'll need to register. With the free registration you can deposite as many images as you want publicly, plus one private image that is only accessible to you. You can buy more space if you want that.

### Manual push

I have made public repository to show you how to deposit custom images on Dockerhub - it's <a href="https://hub.docker.com/repository/docker/chrishah/docker-training-push-demo" target="_blank" >here</a>.

Let's deposit our image there. In order for Dockerhub to know where the image should go I need to rename it to match the name of the repository which is usually something like `username/reponame`. My Dockerhub username is `chrishah`, and I called the repo `docker-training-push-demo`. Note that I will also give the image a specific tag `v07092020`. This could be anything as long as it's in one word an all lower case.
```bash
(host)-$ docker tag automatic_blast_image chrishah/docker-training-push-demo:v07092020
```

Now we can push it Dockerhub.
```bash
(host)-$ docker push chrishah/docker-training-push-demo:v07092020
```

Done! Check it out on <a href="https://hub.docker.com/repository/docker/chrishah/docker-training-push-demo" target="_blank" >Dockerhub</a>.

This image can now be pulled and used by anybody!

```bash
(host)-$ docker run --rm chrishah/docker-training-push-demo:v07092020
```

Also, if you happen to be using `Singularity` rather than `Docker`, this image is compatible. Assuming you have `Singularity` up and running you could just do the following (add `--disable-cache` to pull afresh):
```bash
(host)-$ singularity run docker://chrishah/docker-training-push-demo:v07092020 blastn -h
```

### Automated build

A very neat feature feature in my opinion is that Dockerhub allows you to link its repos to Github repositories. By this, one can neatly and reprodcibly organize one's Docker containers.

Check out this example <a href="https://hub.docker.com/r/chrishah/ncbi-blast" target="_blank" >here</a>.

# Demos

## Running an RStudio server

The demo is inspired by <a href="http://ropenscilabs.github.io/r-docker-tutorial/" title="ropenscilab R Docker tutorial (last accessed 24.04.2020)" target="_blank" >this</a> tutorial and relies on images provided by <a href="https://www.rocker-project.org/" title="Rocker Project Main" target="_blank" >The Rocker Project</a> (see also the <a href="https://github.com/rocker-org/rocker/wiki" title="Rocker Project Github Wiki" target="_blank" >Github Wiki</a>).

Start the RStudio server Docker container like so:
```bash
(host)-$ docker run -e PASSWORD=yourpassword --rm -p 8787:8787 rocker/rstudio:4.0.3
```

Then scoot to `http://localhost:8787` in your webbrowser. Enter your username `rstudio` (per default) and password we've set it to `yourpassword` when we called the container.

If you also want to read/write files on your host from within the container, you can extend the above command, like so, e.g.:
```bash
(host)-$ docker run -d -e PASSWORD=yourpassword -e USERID=$UID --rm -v $(pwd):/working -w /working -p 8787:8787 rocker/rstudio:4.0.3
```

For an example Dockerfile you can use to build an Rstudio image that has some packages already pre-installed, see this [Dockerfile](https://github.com/chrishah/docker-intro/tree/master/Dockerfiles/Dockerfile). Incidentally, this is the setup I used for doing the Differential Expression analyses a few lectures ago.


## Jupyter Notebook

Cyverse US has created a number of Docker images and deposited the contexts on Github <a href="https://github.com/cyverse-vice/" target="_blank" >here</a>.

Very nice are for example their Jupyterlab Servers in Docker containers. Try the following, but note that this image is rather large and may take a while to download, depending on your download speed..
```bash
(host)-$ docker run -it --rm -v /$HOME:/app --workdir /app -p 8888:8888 -e REDIRECT_URL=http://localhost:8888 cyversevice/jupyterlab-scipy:2.2.9
```

Once the download has finished and the server started running move to `http://localhost:8888` in your webbrowser. Cool, no?

Here's another one that has `snakemake` setup within it.
```bash
(host)-$ docker run -it --rm -v /$HOME:/app --workdir /app -p 8888:8888 -e REDIRECT_URL=http://localhost:8888 chrishah/snakemake-vice:v05062020 
```



## Mkdocs server

MkDocs (<a href=https://www.mkdocs.org/" target="_blank" >mkdocs.org</a>) is a neat tool for creating project documentation sites. Instead of installing and running it locally, why not build it into an image and run it from within a Docker container?

To keep things organized, let's first make a new directory.
```bash
(host)-$ mkdir mkdocs && cd mkdocs
```
Open your favorite text editor, copy, paste and save the following text into a file called `Dockerfile` in the `mkdocs` directory you've just created.
```
FROM jfloff/alpine-python:2.7-slim

WORKDIR /usr/src/app

RUN pip install --no-cache-dir mkdocs==0.17.1 Pygments==2.2 pymdown-extensions==3.4

WORKDIR /docs

EXPOSE 8000

ENTRYPOINT ["mkdocs"]
CMD ["serve", "--dev-addr=0.0.0.0:8000"]
```

Now, build your image:
```
(host)-$ docker build -t mkdocs-serve .
```

I've deposited the context for a little test site on Github - let's clone it - guess what, using Docker..
```bash
(host)-$ docker run -ti --rm -v ${HOME}:/root -v $(pwd):/git alpine/git:v2.24.2 clone https://github.com/chrishah/mkdocs-readthedocs-docker-demo.git
```
Then, move into it - `cd mkdocs-readthedocs-docker-demo/`.

Now we're ready to launch our server.

```bash
(host)-$ docker run -it --rm -p 8000:8000 -v ${PWD}:/docs --name mkdocs-serve mkdocs-serve
```

In your webbrowser, scoot to `http://localhost:8000` and see what you have done.

Shut down the running server by pressing `CTRL+C`.

_Note:_ If you cloned the context of the test site using the method above, you might need to change permissions on the directory in case you want to modify it, like so `sudo chown -R $USER:$USER mkdocs-readthedocs-docker-demo/`.


# Links
 - <a href="https://hub.docker.com/" title="Dockerhub" target="_blank">Dockerhub</a>
 - Dockerhub's <a href="https://docs.docker.com/docker-hub/" title="Dockerhub's documentation" target="_blank">documentation</a>
 - The Rocker Project <a href="https://www.rocker-project.org/" title="Rocker Project Main" target="_blank" >Main</a> / <a href="https://github.com/rocker-org/rocker/wiki" title="Rocker Project Github Wiki" target="_blank" >Github Wiki</a>

# Contact

Christoph Hahn - <christoph.hahn@uni-graz.at>
