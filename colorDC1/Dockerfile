FROM jupyter/datascience-notebook:ubuntu-18.04
COPY jupyter_notebook_config.json /opt/conda/etc/jupyter/jupyter_notebook_config.json
#
# Maintained by
LABEL maintainer="Kayla Bieser"
LABEL maintainer_email="kayla.bieser@nsc.edu"
LABEL version="1.0"
LABEL lesson=biol416
#
#
USER root
#
# Un-minimize the image to get back Ubuntu
# goodies for teaching Linux/bash (e.g. man files, etc)
# this makes the container big
RUN apt-get update && apt-get upgrade -y --fix-missing && \
(echo y | DEBIAN_FRONTEND=noninteractive sh -x /usr/local/sbin/unminimize)
RUN apt update -y && apt upgrade -y && apt-get install -y --fix-missing\
 bzip2\
 curl\
 git\
 gnupg2\
 sudo\
 zip\
 unzip\
 wget
#
#
# Install iCommands
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - \
    && echo "deb [arch=amd64] https://packages.irods.org/apt/ bionic main" > /etc/apt/sources.list.d/renci-irods.list \
    && apt-get update \
    && apt-get install -y irods-icommands \
    && apt-get clean \
    && rm -rf /usr/lib/apt/lists/*
#    && fix-permissions $CONDA_DIR
#
## Enable prompt color in the skeleton .bashrc before creating the default NB_USER
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc
#
# Install Jupyter lab
RUN pip install jupyterlab==1.0.9 \
    && conda update -y -n base conda \
    && conda install -c conda-forge nodejs \
    && jupyter lab build
#
#
# install the irods plugin for jupyter lab
RUN pip install jupyterlab_irods==3.0.2 \
    && jupyter serverextension enable --py jupyterlab_irods \
    && jupyter labextension install ijab@3.0.2
#
#
# install jupyterlab hub-extension, lab-manager, bokeh
RUN jupyter lab --version \
    && jupyter labextension install @jupyterlab/hub-extension \
                                    @jupyter-widgets/jupyterlab-manager
#
#
#
# install jupyterlab git extension
RUN jupyter labextension install @jupyterlab/git && \
        pip install --upgrade jupyterlab-git && \
        jupyter serverextension enable --py jupyterlab_git
#
#
# install jupyterlab github extension
RUN jupyter labextension install @jupyterlab/github
#
#
# add Bash kernel
RUN pip install bash_kernel && python3 -m bash_kernel.install
#
#
# Configure conda
RUN conda config --add channels defaults
RUN conda config --add channels bioconda
RUN conda config --add channels conda-forge
#
#
# Install bioconda packages
RUN conda install -y -q fastqc=0.11.7=5
RUN conda install -y -q trimmomatic=0.38=0
RUN conda install -y -q sra-tools=2.9.6=hf484d3e_0
#RUN conda install -y -q parallel-fastq-dump=0.6.5=py_0
#RUN conda install -y -q kallisto=0.46.0=hb6a4e58_0
#RUN conda install -y -q fastp=0.20.0=hdbcaa40_0
RUN conda install -y -q bwa=0.7.17=hed695b0_7
RUN conda install -y -q samtools=1.9=h10a08f8_12
RUN conda install -y -q bcftools=1.8=h4da6232_3
RUN conda install -y -q snpeff=4.3.1t=2
RUN conda install -y -q seqtk=1.3=hed695b0_2
RUN conda install -y -q snpsift=4.3.1t=2
#
#
#
# Genomeview essentials for python
RUN apt update -y && apt-get install -y --fix-missing\
 zlib1g-dev\
 libbz2-dev\
 libcurl4-openssl-dev\
 libssl-dev
RUN /opt/conda/bin/pip install genomeview
COPY ./__init__.py /opt/conda/lib/python3.7/site-packages/genomeview/__init__.py
#
#
# Install nodejs kernel
RUN npm config set user 0 \
 $$ npm install zeromq \
 && npm config set unsafe-perm true \
 && npm install ijavascript -g
#
RUN ijsinstall --hide-undefined --install=global
#
#
# Configure shell
RUN rm /bin/sh && ln -s /bin/bash /bin/sh && echo "SHELL=/bin/bash" >> /etc/environment
# Command that allows colorization of terminal in /home/gea_user, allows recognition of the .bashrc file
RUN cat /etc/skel/.bashrc >> /etc/bash.bashrc
#
#
# Add the GEA user
RUN useradd -p $(openssl passwd -1 -salt xyz 'gea_user') -m -d /home/gea_user -s /bin/bash gea_user
RUN usermod -a -G sudo,root gea_user
RUN echo "gea_user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
#
#
# Switch to GEA user
COPY ./jupyter_notebook_config.json /opt/conda/etc/jupyter/jupyter_notebook_config.json
COPY ./jupyter_notebook_config.py /home/gea_user/.jupyter/jupyter_notebook_config.py
COPY ./.bashrc /home/gea_user/.bashrc
RUN chown -R gea_user:gea_user /home/gea_user/.jupyter
RUN chmod -R 777 /home/jovyan
RUN chown gea_user:gea_user /home/gea_user/.bashrc
USER gea_user
#
#
# Link conda executables to /bin and /usr/lib
RUN sudo ln -s /opt/conda/pkgs/*/bin/* /bin; exit 0
RUN sudo ln -s /opt/conda/pkgs/*/lib/* /usr/lib; exit 0
#
#
#
# Define entry point and startup commands
#ENTRYPOINT ["jupyter"]
#CMD ["lab", "--no-browser", "--ip=0.0.0.0", "--NotebookApp.token=''", "--NotebookApp.password=''", "--notebook-dir='/home/gea_user'", "--Session.username='gea_user'"]
#
COPY ./entry.sh /bin
#
#COPY ./Persistence_Notebook-KB.ipynb /home/gea_user/
#
ENTRYPOINT ["bash", "/bin/entry.sh"]
