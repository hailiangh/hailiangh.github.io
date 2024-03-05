# Installing Vim on your local directory without root

### Clone Vim from GitHub
```
git clone https://github.com/vim/vim.git
cd vim/src
```

### Install in local directory
```
# suppose you want to intall it in ~/.local
./configure --prefix=$HOME/.local
make && make install
```

### Make the installed Vim as default
```
# add the following in ~/.bashrc
export PATH=$HOME/.local/bin/:$PATH
```

### Verify that Vim is installed
```
source ~/.bashrc
which vim
vim --version
```

Reference:
- https://www.vim.org/download.php#unix
- https://superuser.com/questions/162560/how-to-install-vim-on-linux-when-i-dont-have-root-permissions
