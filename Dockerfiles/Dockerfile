FROM rocker/rstudio:4.0.3

RUN apt-get update && \
	apt-get install -y \
		zlib1g-dev \
		libxml2 \
		libbz2-dev \
		liblzma-dev \
		libpcre3-dev \
		libicu-dev 


#install r packages
RUN install2.r --error \
	devtools \
	XML \
	dplyr 

#install R packages via devtools
RUN R -e "devtools::install_github('PF2-pasteur-fr/SARTools@2b95eaa473c20d85f0fa92305acdf43f68f9baed', build_opts='--no-resave-data')"

CMD ["/init"]

