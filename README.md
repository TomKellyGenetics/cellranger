# Building Cell Ranger 2.1.0

This version of cellranger has minor changes for compatibility with current
software releases. It builds cellranger 2.1.0 with minor modifications to
run without errors. This is not an officially supported release but aims
to give the same results as cellranger 2.1.0 for testing pipelines which
require cellranger to be installed.

## Build dependencies
- Python 2.7.17
- rust 1.40.0
- clang 6.0.0
- go 1.9

### Example setup of build dependencies on Ubuntu 18.04.4 LTS (Bionic Beaver)
```
sudo apt-get install make clang-6.0 libz-dev libbz2-dev liblzma-dev

#install go-1.9 from source as not available for Ubuntu 18.04
wget https://dl.google.com/go/go1.9.linux-amd64.tar.gz
tar -xvf go1.9.linux-amd64.tar.gz
sudo mv go /usr/local
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
sudo ln -s /usr/lib/go-1.11/bin/go /usr/bin/go


# Add golang to path
export PATH=/usr/lib/go-1.9/bin:$PATH

# Install rustup from https://www.rustup.rs/ and configure Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh  -s -- -y
sudo apt-get install -y libstd-rust-1.40 cargo
ENV PATH /root/.cargo/bin/:$PATH
ENV PATH  $HOME/.cargo/bin:$PATH
RUN bash $HOME/.cargo/env

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh  -s -- -y

apt-get install -y libstd-rust-1.40 cargo
export PATH=/root/.cargo/bin/:$PATH
export PATH=$HOME/.cargo/bin:$PATH

rustup install 1.40.0
rustup default 1.40.0
```

## Build command
```
git clone https://github.com/TomKellyGenetics/cellranger.git cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001
cd cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001
make && make louvain-clean &&  make louvain
cd -
```

Set up directory structure to match 10x tar release

```
ln -s cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/bin/cellranger cellranger-3.0.2.9001/cellranger \
```

Set up paths to cellranger. This is most of what sourceme.bash would do.
```
export PATH=$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/bin/:$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/lib/bin:$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/tenkit/bin/:$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/tenkit/lib/bin:/martian/bin/:$PATH
export PYTHONPATH=$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/lib/python:$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/tenkit/lib/python:$PWD/martian/adapters/python:$PYTHONPATH
export MROPATH=$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/mro/:$PWD/cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/tenkit/mro/
export _TENX_LD_LIBRARY_PATH=whatever
```

# Running Cell Ranger
## Runtime dependencies
- Binary dependencies can be found in the Ranger 2.1.0 package (https://support.10xgenomics.com/developers/software/downloads/latest)
  - The Ranger package includes a build of the Martian platform (v2.3.0), which is open source. For more information, go to http://martian-lang.org/ .

The dependences can be installed from source:
 
```
# Install node.js
curl -sL https://deb.nodesource.com/setup_13.x | bash - \
 && apt-get install -y nodejs

# Install Martian. Note that we're just building the executables, not the web stuff
RUN git clone --recursive https://github.com/martian-lang/martian.git \
 && cd martian \
 && make mrc mrf mrg mrp mrs mrstat mrjob
 
# Install bcl2fastq. mkfastq requires it.
RUN apt-get update \
 && apt-get install -y alien unzip wget \
 && wget https://support.illumina.com/content/dam/illumina-support/documents/downloads/software/bcl2fastq/bcl2fastq2-v2-19-1-linux.zip \
 && unzip bcl2fastq2*.zip \
 && alien bcl2fastq2*.rpm \
 && dpkg -i bcl2fastq2*.deb \
 && rm bcl2fastq2*.deb bcl2fastq2*.rpm bcl2fastq2*.zip

# Install STAR aligner
RUN wget https://github.com/alexdobin/STAR/archive/2.5.1b.tar.gz \
 && tar xf 2.5.1b.tar.gz \
 && rm 2.5.1b.tar.gz \
 && cd STAR-2.5.1b \
 && make \
 && mv bin/Linux_x86_64_static/STAR* /usr/bin \
 && cd .. \
 && rm -rf STAR-2.5.1b

# Install tsne python package. pip installing it doesn't work
RUN git clone https://github.com/mckinsel/tsne.git \
 && cd tsne \
 && make install \
 && cd .. \
 && rm -rf tsne
``` 


## Setting up the environment
```
# Setup Martian and binary dependencies
source /path/to/ranger/sourceme.bash

# Setup Cell Ranger
source /path/to/cellranger/sourceme.bash
```

## Note about Loupe
The binaries required to generate Loupe Cell Browser (.cloupe) and Loupe V(D)J Browser files (.vloupe) are not included in this repository or in the binary dependencies package Ranger. By default, you will get empty .cloupe/.vloupe files when running a version of Cell Ranger built from this repository. The necessary binaries can be obtained from an existing binary version of Cell Ranger by running:
`cp /path/to/cellranger-2.1.0/cellranger-cs/*/lib/bin/{crconverter,vlconverter} /path/to/open-source-cellranger/lib/bin/`

# Support
We do not provide support for building and running this code.

The officially supported release binaries are available at: (https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest)
