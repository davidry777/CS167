# WSL: Windows Subsystem for Linux

## Prerequisites

* Install [Ubuntu for Windows](https://ubuntu.com/wsl), that page provides detailed instructions for installing it on Windows 10/11.
* Install [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=US).
* Download and install [Notepad++](https://notepad-plus-plus.org/downloads/). We will need to use it to edit text files for WSL.
  * Do not use the default Windows Notepad. It cannot edit line endings.
  * Other text editors will also work, just make sure they can change the line endings for Linux.
  * You don't need text editor if you know how to use Linux text edit commands like `vi` or `nano`.
* Download Oracle JDK 8 for Linux.
  1. Go to [https://www.oracle.com/java/technologies/downloads/#java8-linux](https://www.oracle.com/java/technologies/downloads/#java8-linux)
  2. Download **XXX Compressed Archive**. The current latest version is **8u333**.
  * If your system is 64bit Intel/AMD CPU (x64, x86_64 or AMD64), download **x64 Compressed Archive &rarr; jdk-8u333-linux-x64.tar.gz**.
  * If your system is 32bit Intel/AMD CPU (x86), download **x86 Compressed Archive &rarr; jdk-8u333-linux-i586.tar.gz**.
  * If your system is 64bit ARM CPU (ARM64), download **ARM 64 Compressed Archive &rarr; jdk-8u333-linux-aarch64.tar.gz**.
    * ARM64 CPU may be more commonly seen in tablets, like Surface series.
* Download and install Spark, refer to [lab 5](./CS167-Lab5.md) for detailed instructions.
* Assume you have installed Hadoop and Spark in the following locations:
  * Hadoop: `C:\cs167\hadoop-3.2.2`
  * Spark: `C:\cs167\spark-3.2.1-bin-without-hadoop`
    * You shall still set environment variables for Hadoop and Spark for Windows.

---

## Install Oracle JDK 8 for WSL

Assume you want to install JDK 8 to `~\jdk1.8.0_333` in WSL.

1. Open WSL terminal.
2. Go to your home directory.

    ```bash
    cd ~
    ```

3. Get the absolute path of your home directory, run

    ```bash
    pwd
    ```

    It will print something like `/home/yourname`. Copy this path as we will use it later.

4. Open Windows Explorer for your WSL home directory.

    ```bash
    explorer.exe .
    ```

5. Copy your downloaded JDK 8's tar.gz file, **jdk-8u333-linux-x64.tar.gz** for example, to the opened Windows Explorer window.

6. In WSL Terminal, run

    ```bash
    tar -xf jdk-8u333-linux-x64.tar.gz
    ```

    Once it's done, you shall see a directory **jdk1.8.0_333** created under your WSL home directory. You can either check it via Windows Explorer, or via `ls` command.

---

## Configure WSL environment variables

1. Create **.bashrc** file, run

    ```bash
    touch ~/.bashrc
    ```

    This will create the file if it does not exist.

2. In Windows Explorer, you shall see **.bashrc** file there, open it with **Notepad++**.

3. Add the following lines into **.bashrc**:

    Replace `yourname` with your actual home directory path.

    ```text
    export JAVA_HOME="/home/yourname/jdk1.8.0_333"
    export HADOOP_HOME="/mnt/c/cs167/hadoop-3.2.2"
    export SPARK_HOME="/mnt/c/cs167/spark-3.2.1-bin-without-hadoop"

    PATH=$SPARK_HOME/bin:$HADOOP_HOME/bin:$JAVA_HOME/bin:$PATH
    ```

    Be careful about the letter case and slashes (`/` in Linux vs `\` in Windows). The driver letter shall be lower case after **/mnt**. Also, Linux filesystem is case sensitive, so the path in WSL must be exactly the same as the path in Windows.

    Since we don't use Maven in WSL, we don't set `MAVEN_HOME` here. If you want to use Maven as well, just add one line for `MAVEN_HOME` and add its `bin` folder to `PATH`.

    At this moment, you have two JDK 8 installed. One Windows JDK for Windows use only, one Linux JDK for WSL use only. Hadoop, Spark (and Maven) can be shared by both Windows and WSL.

4. Change the line endings from `CRLF` (`\r\n`) to `LF` (`\n`).
    * In Notepad++'s toolbar, select **Edit**, then select **EOL Conversion**, then select **Unix (LF)**.

5. Save the file.

6. In terminal, check if the file has been edited successfully.

    ```bash
    cat ~/.bashrc
    ```

7. Reload bash environment, run

    ```bash
    source ~/.bashrc
    ```

8. Test 1

    ```bash
    echo $JAVA_HOME
    echo $HADOOP_HOME
    echo $SPARK_HOME
    ```

    They should print correct paths.

9. Test 2

    ```bash
    javac -version
    hadoop version
    spark-submit --version
    ```

    Example output

    ```console
    $ javac -version
    javac 1.8.0_333

    $ hadoop version
    Hadoop 3.2.3
    Source code repository https://github.com/apache/hadoop -r abe5358143720085498613d399be3bbf01e0f131
    Compiled by ubuntu on 2022-03-20T01:18Z
    Compiled with protoc 2.5.0
    From source with checksum 39bb14faec14b3aa25388a6d7c345fe8
    This command was run using /Users/ta/cs167/hadoop-3.2.3/share/hadoop/common/hadoop-common-3.2.3.jar

    $ spark-submit --version
    Welcome to
       ____              __
      / __/__  ___ _____/ /__
     _\ \/ _ \/ _ `/ __/  '_/
    /___/ .__/\_,_/_/ /_/\_\   version 3.2.1
       /_/
                            
    Using Scala version 2.12.15, Java HotSpot(TM) 64-Bit Server VM, 1.8.0_333
    Branch HEAD
    Compiled by user hgao on 2022-01-20T18:44:36Z
    Revision 4f25b3f71238a00508a356591553f2dfa89f8290
    Url https://github.com/apache/spark
    Type --help for more information.    
    ```

    I was running these commands with Hadoop 3.2.3. You may see Hadoop 3.2.2 when you run `hadoop version`.

---

## Start Spark

You can now run thoese `{start, stop}-master.sh` and `{start, stop}-worker.sh` commands on WSL terminal.

For other Spark related commands, you shall still run them in Windows CMD or Windows Terminal.
