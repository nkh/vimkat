## NAME
	vimkat - render a file via Vim
## SYNOPSIS
	vimkat input_file [max_lines] [vimrc]
	... | vimkat [max_lines] [vimrc] [vim_command]   
## DESCRIPTION
	Renders a file via Vim ... quite fast

	input_file	the file you want rendered
	max_lines	the maximum number of lines to render, -1 for all lines
	vimrc		a vimrc file to use instead for your default vimrc, -1 for vimkat's vimrc
	vim_command	extra argument to vim, eg: "-c 'ft=sh'"

Large bash script rendering:
![Screenshot](https://raw.github.com/nkh/vimkat/main/screenshots/time_ftl_full.png)

FZF preview:
![Screenshot](https://raw.github.com/nkh/vimkat/main/screenshots/fzf_preview.png)

The venerable 'vimcat' is often too slow for rendering previews for fzf, specially on large files, and I _REALLY_ want to 
preview my files the _same way_ I'd see them in my $EDITOR (vim of course).

After a short discussion with Rafael Kitover, of vimpager's fame, I decided to dig in, this
is the first beta release of 'vimkat', more releases to fix some bug will come continuously.

## SPEED

Expect at least 3 times faster rendering speed, with an average of ten times and for some files 50! times.

in the example directory you'll find bash_ftl which is a large, very syntax heavy, file which takes 25 seconds
to render (on my machine) with vimcat and 0.5 secons with vimkat.

I'm of course interested in hearing about the performances with other types of files and other machines

### Partial Rendering

if you want took at only the top part of a file you can pass a length argument to 'vimkat' to speed it up.

### No plugins

Loading plugins takes time, you can shave of a tenth of a second or so by using a simplified vimrc file. As a 
third argument 'vimkat' takes a vimrc file or you can pass -1 and it wil use ~/.vimkatrc. If you don't give a
vimrc argument your default vimrc is used.
 
## PIPING

You can pipe in vimkat, you may need to help vim with the file type detection:

	$> generate_text | vimkat [max_lines] [vimrc] [vim_command]   

## PAGER
None yet but it's in the plans

## FZF

My main goal was to have previews in fzf, you can find an example script in the 'fzf' directory.

If you have 'lscolors' installed (https://github.com/sharkdp/lscolors), uncomment the script's
first line to get the files colored too. ls, fd, ... have options to always colorize their output

## HELPING/BUGS

Patches/Bug reports are welcome, and it's easy in such a small project, for:

- examples with your syntax, plus rendering time
- timing data if you find rendering too slow
- wrong rendering
- a logo
- better fzf script
- porting (this works on my machine®)

## AUTHOR/LICENSE
	© Nadim Ibn Hamouda El Khemir, 2022. Artistic License 2.0 or Vim's license
	mailto: nadim.khemir@gmail.com

## SEE ALSO
	vimpager && vimcat (https://github.com/rkitover/vimpager)


