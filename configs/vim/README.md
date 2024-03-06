# My Vim Configurations

## Installing Vim on your local directory without root

### Clone Vim from GitHub
```bash
git clone https://github.com/vim/vim.git
cd vim/src
```

### Install in local directory
```bash
# suppose you want to intall it in ~/.local
./configure --prefix=$HOME/.local
make && make install
```

### Make the installed Vim as default
```bash
# add the following in ~/.bashrc
export PATH=$HOME/.local/bin/:$PATH
```

### Verify that Vim is installed
```bash
source ~/.bashrc
which vim
vim --version
```

Reference:
- https://www.vim.org/download.php#unix
- https://superuser.com/questions/162560/how-to-install-vim-on-linux-when-i-dont-have-root-permissions

## Vim Plugins

### Using Vundle as the plugin manager

### [NerdTree: A file system explorer for the Vim editor](https://github.com/preservim/nerdtree)  

### [Solarized Colorscheme for Vim](https://github.com/altercation/vim-colors-solarized)
```bash
git clone https://github.com/altercation/vim-colors-solarized.git
cd vim-colors-solarized/colors
mv solarized.vim ~/.vim/colors/

## Put the following in ~/.vimrc
syntax enable
set background=dark
colorscheme solarized

## or for the light background
syntax enable
set background=light
colorscheme solarized

## or different background in GUI and terminal modes
if has('gui_running')
    set background=light
else
    set background=dark
endif
```


