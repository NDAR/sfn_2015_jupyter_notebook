### Set up Jupyter and SciPy stack

Configured using the AWS 64 bit base Ubuntu (14.04) AMI ( mi-d05e75b8).

This notebook is somewhat simplified from the version demonstrated at our 2015 Society for Neuroscience Symposia in that it does not include the example integrations with R.  
The SfN version of the notebook is available, just write to the NDA Help Desk: NDAHelp at mail.nih.gov.

This version of the notebook includes some additions, like free-text query of VCF files and API access to VCF files using pyvcf with the vcf files being streamed from S3 objects.

Note you should have an SSH client that supports X11 forwarding, which is how the notebook will be run; remotely inside a browser on the AWS EC2 instance.

#### Ubuntu instructions
##### Download and install a browser
###### Chrome
    ```
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
    sudo apt-get update
    sudo apt-get install -y google-chrome-stable
    ```

###### Firefox

    ```
    sudo apt-get -y install firefox
    ```

##### Build Tools and Utilities

    ```
    sudo apt-get install -y build-essential git dbus-x11 libaio-dev unzip r-base npm nodejs-legacy
    ```

##### Python Dependencies

    ```
    sudo apt-get -y install python-dev python3-numpy python3-scipy python3-pip python3-pandas
    sudo pip3 install cython jupyter boto ipython-sql pyvcf
    ```

### Setup cx_Oracle

#### Download Oracle client libraries and configure environment variables

Step by step instructions for doing this manually.  This Jupyter Notebook was written using the  version 11.2.

1. Download the Instant Client Package - Basic and Instant Clieent Package - SDK zip files from http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html

2. Extract both zip files to /usr/lib/Oracle/ and create symbolic link for library

    ```
    sudo mkdir /usr/lib/oracle
    sudo unzip instantclient-basic-linux.x64-11.2.0.4.0.zip -d /usr/lib/oracle
    sudo unzip instantclient-sdk-linux.x64-11.2.0.4.0.zip -d /usr/lib/oracle
    sudo ln -s /usr/lib/oracle/instantclient_11_2/libclntsh.so.11.1 /usr/lib/oracle/instantclient_11_2/libclntsh.so
    ```

3. Set the ORACLE_HOME environment variable to the folder containing the unzippped files.

    ```
    export ORACLE_HOME=/usr/lib/oracle/instantclient_11_2
    ```

4. Create a drop-in file in /etc/profile.d/ to automatically set environment variables whenever a user logs onto a shell.

Use the ORACLE_HOME directory set in step 4 to set the ORACLE_HOME variable, and then add this to LD_LIBRARY_PATH and PATH environment variables.

    ```
    sudo vim /etc/profile.d/oracle.sh
    export ORACLE_HOME=/usr/lib/oracle/instantclient_11_2
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
    export PATH=$PATH:"$ORACLE_HOME/bin"
    ```

5. Source the newly created drop-in file so your environment variables are set. Note, these need to be set for the root user as well to install to /usr/lib.

    ```
    source /etc/profile.d/oracle.sh
    ```

#### Download cx_Oracle and configure

1. Download the source files from Source Forge

    ```
    wget http://sourceforge.net/projects/cx-oracle/files/5.1.2/cx_Oracle-5.1.2.tar.gz
    ```

2. Extract to a shared location (i.e., /usr/lib)

    ```
    sudo tar -xvf cx_Oracle-5.1.2.tar.gz -C /usr/lib
    ```

3. Run setup.py file for cx_Oracle

You will probably have to run this as root and source /etc/profile.d/oracle.sh if you are using these instructions. 

    ```
    sudo su
    cd /usr/lib/cx_Oracle-5.1.2
    source /etc/profile.d/oracle.sh
    python3 setupy.py install
    ```

#### Test

1. Open python console
2. import cx_Oracle, if you get an error message related to shared libraries then likely your LD_LIBRARY_PATH does not include the folder where you have extracted the instantclient zip files.

### Clone aws_token_generator and setup

1. Clone the repository for NIMH Data Archives github repository

    ```
    cd ~
    git clone https://github.com/NDAR/nda_aws_token_generator.git
    ```

2. Install the python package.

    ```
    cd nda_aws_token_generator/python3/
    sudo python3 setup.py install
    ```
### Clone and Build samtools with S3 support

See https://www.biostars.org/p/147772/

1. Make sure to install libcurl.

    ```
    sudo apt-get install libcurl4-openssl-dev
    ```
2. Clone and Build htslib and samtools.
    
    ```
    cd ~
    git clone https://github.com/DonFreed/htslib.git -b libcurl
    cd htslib
    git checkout -b libcurl
    autoconf
    ./configure --enable-libcurl
    sudo make install
    cd ..
    git clone https://github.com/samtools/samtools.git -b 1.2
    cd samtools
    git checkout -b 1.2
    sudo make install LDLIBS+=-lcurl LDLIBS+=-lcrypto    
    ```
    
### Run Jupyter Notebook

1.  Jupyter Notebook

    ```
    cd ~
    jupyter notebook
    ```

2.  If a browser is not automatically launched, launch a browser and open localhost:{port} where {port} is the port specified by Jupyter  after running the previous command.
3.  Open the notebook file included as part of this repostiory (i.e., http://localhost:8888/).