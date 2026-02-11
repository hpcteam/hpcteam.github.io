# **Python Installation**

This method is particularly useful in offline environments or when you require a specific Python version that may not be available in your package manager.

---

## **Step 1: Download the Tar File**

1. Visit the official Python website at [https://www.python.org/downloads/](https://www.python.org/downloads/).
2. Navigate to the **Downloads** section.
3. Choose the desired Python version and select the corresponding **Gzipped source tarball**.

---

## **Step 2: Extract the Tar File**

Once the tar file is downloaded, follow these steps:

1. Navigate to the directory where the file is saved.
2. Open a terminal or command prompt.
3. Use the `tar` command to extract the contents of the tar file:

```
tar xf Python-3.14.3.tar.xz
```
![extracting pyhton](/assets/Python/Python-extract.png)

* **Explanation**: The `tar` command extracts the `.tar.xz` file.

  * `x`: Extract the contents of the archive.
  * `f`: Specify the file name.

---

## **Step 3: Configure the Installation**

1. Navigate into the extracted directory:

```
cd Python-3.14.3
```

2. Run the `configure` script to prepare the installation. This step ensures that the necessary dependencies are detected and sets up the installation according to your system’s configuration. You can use the `--prefix` argument if you need to install Python in a custom directory:

```
./configure --prefix=/home/softwares/Python-3/3.14.3 --enable-shared --enable-optimizations
```
![configure pyhton](/assets/Python/configure.png)
   * **Explanation**:

     * `--prefix=/home/softwares/Python-3/3.14.3`: Specifies the installation directory.
     * `--enable-shared`: Builds shared libraries, useful for linking with other programs.
     * `--enable-optimizations`: Optimizes the build for better performance.

---

## **Step 4: Build and Install Python**

After configuring the installation, you can proceed with the build process:

1. **Build Python**:

```
make 
or 
make -j 16 (16 cores)
or 
make -j (avabilabe core)
```
![make pyhton](/assets/Python/make.png)

   * **Explanation**: The `make` command compiles the source code into executable binaries.

2. **Run Tests (Optional but Recommended)**:

   To test the Python interpreter after building, use:

```
make test
```
![make test](/assets/Python/make-test.png)

   * **Explanation**: The `make test` command runs Python’s built-in test suite to ensure everything is working as expected. If tests fail, you may need to investigate the issue.

3. **Install Python**:

   Once the tests are successful, install Python:

```
make install
```
![make install ](/assets/Python/make-install.png)

   * **Explanation**: The `make install` command copies the compiled Python files into the system’s directories. You’ll likely need `sudo` to install Python system-wide.

---

## **Step 5: Verify the Installation**

To verify that Python was installed correctly:
![make install ](/assets/Python/verifying-installation.png)

1. Set up the environment variables:

```
export PATH=/home/softwares/Python/3.14.3/bin:$PATH
export LD_LIBRARY_PATH=/home/softwares/Python/3.14.3/lib:$LD_LIBRARY_PATH
export CPATH=/home/softwares/Python/3.14.3/include:$CPATH
```

![exporting  ](/assets/Python/Exportpyhton.png)

2. Check the Python installation:

```
which python3
python3 --version
```
![exporting  ](/assets/Python/Veriosn check.png)

   * **Explanation**:

     * `which python3`: Shows the path to the installed Python binary.
     * `python3 --version`: Prints the installed Python version.

You should see the installed Python version printed on the screen.

---

## **Step 6: Create a Module File**

To manage multiple versions of Python, it's useful to create a **module file** for loading the Python environment. This is especially helpful if you have multiple versions installed on a shared system or cluster.

---



