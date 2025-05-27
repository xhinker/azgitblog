# Install Python3.11 for NVidia Jetson Nano

NVidia Jetson Nano is an amazing small device with CUDAs, unlike GPUs in desktop machine, which will use up to hundreds of watt in running, this little one used up to 5w based on my experience. 

While this device is cool and good for small AI project, its closed ecosystem comes with only Python 3.6, and lack lots of newer features. I decided to upgrade its Python to 3.11 and install CUDA on it. 

1. Start by updating your system to ensure all packages are current:

```sh 
sudo apt update && sudo apt upgrade
```

2. Get the necessary build scripts by cloning the repository:

```sh
git clone https://github.com/JetsonHacksNano/build_python.git
```

3. Compile Python 3.11, which may take around 2 hours:

```sh
cd build_python
bash ./build_python3.sh
```
The built files will be in `~/Python_Builds`.

4. Set up a local repository for the built Python:

```sh
bash ./make_apt_repository.sh
```
This places `.deb` files in `/opt/debs` and sets up necessary files.

5. Update APT System and Install Python 3.11

```sh
sudo apt update
sudo apt install python3.11-full
```

6. Ensure pip is installed and updated

```sh
python3.11 -m ensurepip --upgrade
```

If get problem here, ensure add `$HOME/.local/bin:$PATH` to PATH. in `~/.bashrc` or `~/.zshrc` append the following to the file:

```ini
export PATH="$HOME/.local/bin:$PATH"
```

then, `source ~/.bashrc` or `source ~/.zshrc` . 

7. Create VENV

```sh
python3.11 -m venv nano_py311_venv
```

