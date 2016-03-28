# Radcli installation

1. Create a folder to contain downloaded software sources

```
mkdir /usr/local/src/radcli
```

2. Move to radcli folder

```
cd /usr/local/src/radcli/
```

3. Download radcli libraries

```
wget https://github.com/radcli/radcli/releases/download/x.x.x/radcli-x.x.x.tar.gz
```

4. Decompress the archive

```
tar zxvf radcli-x.x.x.tar.gz
```

5. Move to radcli-x.x.x folder

```
cd radcli-x.x.x
```

6. Compile and install

```
./configure --sysconfdir=/etc/ && make && make install
```

That's it. Radcli is now installed. remember to keep a copy of source package in 
your /usr/local/src/radcli/ folder, so that you can easily remove the software if needed.
If you need to uninstall, you can do it with the following commands:

```
cd /usr/local/src/radcli/radcli-x.x.x
make uninstall
```
