# Install Vim on your local directory without root

### Clone and compile Vim
```
git clone https://github.com/vim/vim.git
cd vim/src
make
```

### Install in local directory
```
# suppose you want to intall it in ~/.local
make install DESTDIR=~/.local
```

### Make the installed Vim as default
```
# add the following in ~/.bashrc
# verify that vim is installed in the following dir
export PATH=~/.local/usr/local/bin/:$PATH
```

### Verify that Vim is installed
```
which vim
vim --version
```

Reference:
- https://www.vim.org/download.php#unix
- https://superuser.com/questions/162560/how-to-install-vim-on-linux-when-i-dont-have-root-permissions
