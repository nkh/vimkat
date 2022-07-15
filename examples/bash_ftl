#!/bin/env bash

ftl() # dir[/file], pfs, preview_ftl. © Nadim Khemir 2020-2022, Artistic licence 2.0
{
tab=0 ; tabs+=("$PWD") ; ntabs=1 ; : ${prev_all:=1} ; : ${pdir_only[tab]:=} ; : ${find_auto:=README} ; max_depth[tab]=1 ; : ${zoom:=0} ; zooms=(70 50 30) ; mh='Creating montage ...'
tbcolor 67 67 ; quick_display=512 ; cursor_color='\e[7;34m' ; : ${imode[tab]:=0} ; lmode[tab]=0 ; : ${show_line:=1} ; show_size=0 ; show_date=1 ; : ${etag:=0} ; flips=(' ' ' ')
ifilter='webp|jpg|jpeg|JPG|png|gif'; mfilter='mp3|mp4|flv|mkv'; : ${sort_type0:=0} ; sort_filters=(-k3 -n -k2)
fzf_opt="-p 80% --cycle --reverse --info=inline --color=hl+:214,hl:214" ;  fzfp_opt="+m --cycle --expect=ctrl-t --reverse --header='$PWD'"
sglyph=( ⍺ 🡕 ) ; iglyph=('' ᴵ ᴺ) ; lglyph=('' ᵈ ᶠ) ; tglyph=('' ¹ ² ³ D) ; ftl_cfg="$HOME/.config/ftl" ; pgen="$ftl_cfg/generators" ; : ${shell_file:=0} ; auto_tags=1 ; pdhl=

declare -A -g dir_file mime pignore lignore tail tags ntags ftl_env ; export ftl_root=$ftl_cfg/var ; ftl_cmds=$ftl_root/cmds ; ghist=$ftl_root/history ; RM="rm -rf"
mkapipe 4 5 6 ; echo -en '\e[?1049h' ; stty -echo ; my_pane=$(pid_2_pane $$) ; thumbs=$ftl_root/thumbs ; mkdir -p $thumbs ; $pushd "$dir" &>/dev/null ; dir_done=56fbb22f2967 

[[ "$1" ]] && { [[ -d "$1" ]] && { dir="$1" ; search='' ; } || { [[ -f "$1" ]] && { path "$1" ; dir="${p}" ; search="$f" ; } ; } || { echo ftl: \'$1\', no such path ; exit 1 ; } ; }
[[ "$2" ]] && { fs=$2/$$ ; pfs=$2 ; mkdir -p $fs ; touch $fs/history ; } || { fs=$ftl_root/$$ ; pfs=$fs ; main=1 ; mkdir -p $fs/prev ; touch $ghist ; echo $my_pane >$fs/pane ; } ;
fsp=$pfs/prev ; . ~/.ftlrc || . ~/.config/ftl/ftlrc ; marks[$"'"]="$(tail -n1 $ghist)" ; PPWD="$dir" ; rfilters[tab]=$rfilter0 
[[ "$3" ]] && { gpreview=1 ; prev_all=0 ; emode=0 ; prev_synch ; true ; } || { tag_synch ; cdir "$dir" "$search" ; }

while : ; do tag_synch ; winch ; { [[ "$R" ]] && { REPLY="${R:0:1}" ; R="${R:1}" ; } || read -sn 1 -t 0.3 ; } && try bindings ; kbdf ; winch=1 ; REPLY= ; done
}

bindings()
{
pdh "$REPLY\n"
ext_bindings || case "${REPLY: -1}" in
	${C[hexedit]})		tcpreview ; ctsplit "hexedit ${n@Q}" ;;
	${C[help]})		<~/.config/ftl/help fzf-tmux $fzf_opt --tiebreak=begin --header="$(echo -e "\t\t\t")""⇑: alt-gr, ⇈: shift+alt-gr, ˽: leader" ; list ;;
	h|D)			[[ "$PWD" != / ]]  && { nd="${PWD%/*}" ; cdir "${nd:-/}" "$(basename "$p")"; } ;;
	j|B|k|A)		((nfiles)) && { [[ $REPLY == j || "$REPLY" == B ]] && { move 1 && list ; true ; } || { move -1 && list ; } ; } ;;
	l|C|'')			((nfiles)) && { [[ -f "${files[file]}" ]] && { [[ $REPLY == '' ]] && edit ; true ; } || cdir "${files[file]}" ; } ;;
	5|6)			[[ $REPLY == 5 ]] && { move -$LINES && list ; true ; } || { move $LINES && list ; } ;;
	${C[top_file_bottom]})	twd="${tab}_$PWD" ; [[ -f "$n" ]] && { (($file == $nfiles - 1)) && dir_file[$twd]=0 || dir_file[$twd]=$nfiles ; } || dir_file[$twd]=$first_file ; list ;;
	${C[goto_alt1]})	to_search=".$e" ; R="${C[find_next]}" ;;
	${C[goto_alt2]})	for ((i=file + 1 ; i != $nfiles ; i++)) ; do [[ ! "${files[i]##*/}" =~ ".$e" ]] && { list $i ; return ; } ; done ; list ;;
	${C[preview_down]})	((in_vipreview)) && tmux send -t $pane_id C-D ;;
	${C[preview_up]})	((in_vipreview)) && tmux send -t $pane_id C-U ;;
	1|2|3|4)		[[ ${tags[$n]} == ${tglyph[$REPLY]} ]] && unset -v "tags[$n]" || tags[$n]=${tglyph[$REPLY]} ; ((stagsi++)) ; move 1 ; list ;;
	${C[pdh]}) 		pdh_show ;;
	${C[cd]})		prompt 'cd: ' ; [[ -n $REPLY ]] && cdir "${REPLY/\~/$HOME}" || list ;;
	${C[chmod_x]})		for i in "${selection[@]}" ; do [[ -x "$i" ]] && mode=a-x || mode=a+x ; chmod $mode "$i" ; done ; cdir ;;
	${C[clear_filters]})	filters[tab]= ; filters2[tab]= ; ntfilter[tab]= ; filter_rst ; ftag= ; cdir ;;
	${C[command_prompt]})	echo -ne "\e[H" ; tput cnorm ; stty echo ; C=$(cmd_prompt) ; stty -echo ; tput civis ; [[ "$C" ]] && header '' "$head" && shell_command "$C" || list ;;
	${C[command_mapping]})	for k in "${!C[@]}" ; do printf "%-17s %-5s\t\n" $k ${shortcuts[$k]} ; done | sort | column -c 150 | fzf-tmux $fzf_opt --tiebreak=begin ; list ;;
	${C[copy]})		prompt 'cp to: ' && [[ $REPLY ]] && { tag_check && cp_mv_tags p "$REPLY" || cp_mv p "$REPLY" "${selection[@]}" ; } ; cdir ;;
	${C[copy_clipboard]})	printf "%q " "${selection[@]}" | xsel -b -i ;;
	${C[create_bulk]})	tcpreview ; : >$fs/bc ; $EDITOR $fs/bc && perl -i -ne 'if(m-^.$i-){ if(m-/$-){ print "mkdir $_" }else{ print "touch $_" }}' $fs/bc && bash $fs/bc ; cdir ;;
	${C[create_dir]})	prompt 'mkdir: ' && [[ "$REPLY" ]] && mkdir -p "$PWD/$REPLY" && cdir "$PWD/$REPLY" || list ;;
	${C[create_file]})	prompt 'touch: ' && [[ "$REPLY" ]] && touch "$PWD/$REPLY" ; cdir "$PWD" "$REPLY" ;;
	${C[delete]})		tag_check && { delete_tag ; true ; } || delete '' "${selection[@]}" ;;
	${C[depth]})		prompt 'depth: ' && [ "$REPLY" -eq "$REPLY" ] 2>&- && max_depth[tab]=$REPLY && cdir '' "$f" ;;
	${C[editor_detach]})	((in_vipreview)) && in_vipreview= && pane_id= && cdir ;;
	${C[external_mode1]})	emode=1 ; preview ;;
	${C[external_mode2]})	emode=2 ; preview ;;
	${C[external_mode3]})	emode=3 ; preview ;;
	${C[external_mode4]})	emode=4 ; preview ;;
	${C[file_dir_mode]})	((lmode[tab]--)) ; ((lmode[tab] < 0)) && lmode[tab]=2 ; cdir '' "$f" ;;
	${C[filter]})		prompt "filter: " -i "${filters[tab]}" ; filters[tab]="$REPLY" ; ftag="~" ; cdir '' "$f" ;;
	${C[filter2]})		prompt "filter2: " -i "${filters2[tab]}" ; filters2[tab]="$REPLY" ; ftag="~" ; cdir '' "$f" ;;
	${C[filter_ext]})	p=~/.config/ftl/external_filters ; file=$(cd "$p" ; fd | fzf-tmux -p 50% --cycle) ; [[ $file ]] && . "$p/$file" $fs ; cdir '' "$f";;
	${C[filter_reverse]}) 	prompt "rfilter: " -i "${rfilters[tab]}" ; rfilters[tab]="$REPLY" ; ftag="~" ; cdir '' "$f";;
	${C[find]})		prompt "find: " -i to_search ; R="${C[find_next]}" ;;
	${C[find_next]})	for ((i=file + 1 ; i != $nfiles ; i++)) ; do [[ "${files[i]##*/}" =~ "$to_search" ]] && { list $i ; return ; } ; done ; list ;;
	${C[find_previous]})	for ((i=file - 1 ; i != -1 ; i--)) ; do [[ "${files[i]##*/}" =~ "$to_search" ]] && { list $i ; return ; } ; done ; list ;;
	${C[find_fzf_all]})	exec 2>&9 ; tcpreview ; fzf_go "$(fd -HI -E'.git/*' | fzf_vpreview $fzfp_opt)" ; exec 2>"$fs/error_log" ;;
	${C[find_fzf_dirs]})	exec 2>&9 ; tcpreview ; fzf_go "$(fd -td -I -L | fzf_vpreview $fzfp_opt)" ; exec 2>"$fs/error_log" ;;
	${C[find_fzf]})		exec 2>&9 ; tcpreview ; fzf_go "$({ fd -HI -d1 -td | sort ; fd -HI -d1 -tf -tl | sort ; } | fzf_vpreview $fzfp_opt)" ; exec 2>"$fs/error_log" ;;
	${C[gmark]})		{ cat $ftl_root/marks 2>&- ; [[ -d "$n" ]] && echo "$n/\$" || echo "$n" ; } | awk '!seen[$0]++' | sponge $ftl_root/marks ;;
	${C[gmark_fzf]})	[[ -e $ftl_root/marks ]] && fzf_go "$(cat $ftl_root/marks | lscolors | fzf-tmux $fzf_opt --cycle --expect=ctrl-t --ansi)" ;;
	${C[gmarks_clear]})	prompt 'clear persistent marks? [y|N]' -sn1 && [[ $REPLY == y ]] && :>$ftl_root/marks ; list ;;
	${C[go_fzf]})		exec 2>&9 ; tcpreview ; fzf_go "$(fzfppv --expect=ctrl-t)" ; exec 2>"$fs/error_log" ;;
	${C[go_rg]})		exec 2>&9 ; tcpreview ; fzf_rg_go "$(fzfr "fzf--expect=ctrl-t")" ; exec 2>"$fs/error_log" ;;
	${C[history]})		h=$fs/history ; dedup $h && fzf_go "$(<$h lscolors | fzf-tmux $fzf_opt --tac --ansi --expect=ctrl-t)" ;;
	${C[ghistory]})		h=$ghist ; dedup $h && fzf_go "$(<$h lscolors | fzf-tmux $fzf_opt --tac --ansi --expect=ctrl-t)" ;;
	${C[ghistory2]})	h=$ghist ; dedup $h && fzf_go "$(<$h lscolors | fzf-tmux $fzf_opt --tac --ansi --expect=ctrl-t)" ;;
	${C[ghistory_clear]})	prompt 'clear global history? [y|N]' -sn1 && [[ $REPLY == y ]] && rm $ghist 2>&- ; list ;;
	${C[ghistory_edit]})	dedup $ghist && rg -v -x -F -f <(<$ghist lscolors | fzf-tmux $fzf_opt --tac -m --ansi) $ghist | sponge $ghist ;;
	${C[image_fzf]})	exec 2>&9 ; tcpreview ; fzf_go "$(fzfi --expect=ctrl-t -q "$(echo "$ifilter" | perl -pe 's/(^|\|)/ $1 ./g')")" ; exec 2>"$fs/error_log" ;;
	${C[image_mode]})	((imode[tab]--)) ; ((imode[tab] < 0)) && imode[tab]=2 ; ftl_imode ; cdir '' "$f";;
	${C[link]})		tag_check && prompt "Link (${#tags[@]})? [y|N]" -sn1 ; [[ $REPLY == y ]] && tags_clear && for f in "${selection[@]}" ; do ln -s -b "$f" . ; done ; cdir ;;
	${C[mark_fzf]})		fzf_go "$(printf "%s\n" "${marks[@]}" | sort -u | lscolors | fzf-tmux $fzf_opt --expect=ctrl-t --ansi)" ;;
	${C[mark_go]})		read -n 1 ; [[ -n ${marks[$REPLY]} ]] && { dir="$(dirname "${marks[$REPLY]}")" ; cdir "$dir" '' "${dir_file[${tab}_$dir]}" ; } || list ;;
	${C[mark_go_tab]})	read -n 1 ; [[ -n ${marks[$REPLY]} ]] && { dir="$(dirname "${marks[$REPLY]}")" ; tab_new "$dir" ; cdir "$dir" '' "${dir_file[${tab}_$dir]}" ; } || list ;;
	${C[mark]})		read -sn1 ; [[ -n $REPLY ]] && marks[$REPLY]="${files[file]}" ;;
	${C[montage_clear]})	[[ -d "$n" ]] && { rm "$ftl_root/montage/$n/montage.jpg" 2>&- ; pw3image FTL_RESTART_W3M ; list ; } ;;
	${C[montage_preview]})	[[ "$montage" ]] && montage= || montage="⠶" ; list ;;
	${C[mplayer_kill]})	mplayer_k ;;
	${C[pane_down]})	pane_extra "-v" "" ;;
	${C[pane_left]})	pane_extra "-h -b" '' ;;
	${C[pane_right]})	pane_ftl "'$n' $pfs" "-h -b" -R ; [[ "$n" ]] && rdir "$n" ; echo -e "\e[H\e[K" ; header '2' "$head" ;;
	${C[pane_R]})		pane_extra "" "" ;;
	${C[pane_L]})		pane_extra "-h -b" "-R" ;;
	${C[pane_next]})	p=$(pane_next) ; [[ $p ]] && { echo -e "\e[H\e[K" ; header '2' "$head" ; } && tmux selectp -t $p &>/dev/null && tmux send -t $p ${C[refresh]} ;;
	${C[preview]})		((prev_all ^= 1)) ; tcpreview ; sleep 0.05 ; cdir ;;
	${C[preview_once]})	preview=1 ; tcpreview ; sleep 0.05 ; cdir ;;
	${C[preview_dir_only]}) [[ "${pdir_only[tab]}" ]] && pdir_only[tab]= || pdir_only[tab]='ᴰ' ; list ;;
	${C[preview_ext_ign]})	[[ $e ]] && { ((pignore[${e}] ^= 1)) ; cdir ; } ;;
	${C[preview_hide_ext]})	[[ $e ]] && ((lignore[${e}] ^= 1)) ; cdir ;;
	${C[preview_hide_clr]})	lignore=() ; cdir ;;
	${C[preview_image]})	((no_image_preview ^= 1)) ; for e in $(tr '|' ' ' <<< "$ifilter") ; do pignore[${e}]=$no_image_preview ; done ; ((zoom == 0)) && R=${C[refresh]} ; cdir ;;
	${C[preview_lock_clr]})	rm "$fs/lock_preview/$n" 2>&- ; list ;;
	${C[preview_lock]})	p=~/.config/ftl/lock_preview ; file=$(cd $p ; fd | fzf-tmux $fzf_opt) ; [[ $file ]] && . $p/$file ; list ;;
	${C[preview_m1]})	extmode=1 ; list ;;
	${C[preview_m2]})	extmode=2 ; list ;;
	${C[preview_show]})	[[ $e =~ $mfilter ]] && { (~/.config/ftl/viewers/cmus "${selection[@]}" & ) ; ((${#tags[@]})) && { tags_clear ; list ; } ; } ;;
	${C[preview_show_fzf]})	p=~/.config/ftl/viewers ; viewer=$(cd $p 2>&- && fd | fzf-tmux -p80% --cycle --reverse --info=inline) ; [[ $viewer ]] && . $p/$viewer ; list ;;
	${C[preview_size]})	((zoom += 1, zoom >= ${#zooms[@]})) && zoom=0 ; zoom ; [[ $pane_id ]] && tmux resizep -t $pane_id -x $x &>/dev/null ; rdir '' 0 ;;
	${C[preview_tail]})	[[ "${tail[$n]}" == "+$ " ]] && tail[$n]='+0 ' || tail[$n]='+$ ' ; list ;;
	${C[quit]})		tab_close || pane_close || quit ;;
	${C[quit_keep_shell]})	tab_close || pane_close || quit ;;
	${C[quit_all]})		((main)) && { pane_send "${C[quit]}" ; sleep 0.05 ; R=${C[quit]} ; in_Q=1 ; } || tmux send -t $main_pane ${C[quit_all]}  ;;
	${C[quit_keep_zoom]})	quit2 ; [[ $pane_id ]] && { echo >$pfs/pane ; tmux selectp -t $pane_id ; tmux resizep -Z -t $pane_id ; } ; exit 0 ;;
	${C[refresh]})		((nfiles)) && path "${files[file]}" || f= ; tag_check ; cdir "$PWD" "$f" ;; # directory content change signal
	${C[rename]})		tag_check && bulkrename || { prompt "$f » " -i "$f" && [[ $REPLY && "$f" != "$REPLY" ]] && mv "${files[file]}" "$REPLY" && f="$REPLY" ; } ; cdir '' "$f" ;;
	${C[shell]})		{ [[ "$shell_id" ]] && $(tmux has -t $shell_id 2>&-) ; } || shell_pane ; tmux selectp -t $shell_id &>/dev/null ;;
	${C[shell_file]})	for i in "${selection[@]}" ; do shell_send "'$i'" " " ; done ;;
	${C[shell_view]})	tcpreview ; echo -en '\e[?1049l' ; read -sn 1 ; echo -en '\e[?1049h' ; list ;;
	${C[shell_synch]})	shell_send "cd '$p'" C-m ;;
	${C[shell_zoomed]})	{ [[ $shell_id ]] && $(tmux has -t $shell_id 2>&-) ; } || shell_pane ; tmux selectp -t $shell_id &>/dev/null ; tmux resizep -Z -t $shell_id &>/dev/null ;;
	${C[show_etag]})	((etag^=1)) ; cdir ;;
	${C[show_hidden]})	((show_hidden[tab])) && show_hidden[tab]= || show_hidden[tab]=1 ; cdir '' "$f" ;;
	${C[show_size]})	((show_size ^= 1)) || { ((show_size ^= 1)) ; ((show_dir_size ^= 1)) ; } || show_size=0 ; cdir ;;
	${C[show_stat]})	((show_stat ^= 1)) ; refresh ; list ;;
	${C[sort_reverse]})	[[ ${reversed[tab]} == '-r' ]] && reversed[tab]=0 || reversed[tab]=-r ; cdir '' "$f";;
	${C[sort]})		((sort_type[tab] = sort_type[tab] + 1 >= ${#sort_filters[@]} ? 0 : sort_type[tab] + 1)) ; cdir '' "$f" ;;
	${C[tab_new]})		[[ -d "$f" ]] && tab_new "$n" || tab_new "$PWD" ; cdir ${tabs[tab]} ;;
	${C[tab_next]})		ctab=$tab ; tab_next ; ((tab != ctab)) && cdir ${tabs[tab]} '' "${dir_file[${tab}_${tabs[tab]}]}";;
	${C[tag_all_files]})	for p in "${files[@]}" ; do [[ -f "$p" ]] && tags["$p"]='▪' ; done ; list ;;
	${C[tag_all]})		for p in "${files[@]}" ; do tags["$p"]='▪' ; done ; list ;;
	${C[tag_copy]})		tag_check && cp_mv_tags $REPLY "$PWD" ; cdir '' "$f";;
	${C[tag_moveto]})	fzf_mv "${selection[@]}" && tags_clear ;  cdir '' "$f";;
	${C[tag_move]})		tag_check && cp_mv_tags $REPLY "$PWD" ; cdir '' "$f";;
	${C[tag_flip]})		tag_flip "${files[file]}" ; move 1 ; list ;;
	${C[tag_flip_up]})	tag_flip "${files[file]}" ; move -1 ; list ;;
	${C[tag_fzf]})		fzf_tag T "$(fd -H --color=always -d1 | fzf-tmux $fzf_opt -m --ansi --marker '▪')" ; list ;;
	${C[tag_fzf_all]})	fzf_tag T "$(fd -H --color=always | fzf-tmux $fzf_opt -m --ansi --marker '▪')" ; list ;;
	${C[tag_goto]})		tag_check && fzf_go "$(printf "%s\n" "${!tags[@]}" | sort -u | lscolors | fzf-tmux $fzf_opt --tac --ansi --expect=ctrl-t)" ;;
	${C[tag_merge]})	p=~/.config/ftl/merge ; file=$(cd $p 2>&- && fd | fzf-tmux --header 'Merge tags:' $fzf_opt) ; [[ $file ]] && . $p/$file ; cdir ;;
	${C[tags_merge_all]})	source ~/.config/ftl/merge/all ; list ;;
	${C[tag_untag_all]})	tags_clear ; list ;;
	${C[tag_untag_fzf]})	tag_check && { fzf_tag U "$(printf "%s\n" "${!tags[@]}" | lscolors | fzf-tmux $fzf_opt -m --ansi --marker '⊟')" ; list ; } ;;
	${C[tag_external]})	p=~/.config/ftl/external_tags ; { fe=$(cd $p ; fd | fzf-tmux -p 50% --reverse --info=inline) ; [[ $fe ]] && { . $p/$fe $fs ; etag=1 ; cdir ; } ; } ;;
	${C[SIG_PANE]})		pane_read ; ((${#panes[@]})) && tmux selectp -t ${panes[0]} || tmux selectp -t $my_pane ; ofs= ; R=${C[refresh]} ;;
	${C[SIG_REFRESH]})	((gpreview)) && kbdf && prev_synch ;;
	${C[SIG_REMOTE]})	((prev_all)) && { read op <$fsp/pane ; prev_synch prev ; tmux selectp -t $op ; kbdf ; } ;;
	${C[SIG_SYNCH_SHELL]})	IFS=$'\n' read -r shell_dir <$pfs/synch_with_shell ; cdir "$shell_dir" ;;
esac
}

cdir() { inotify_k ; get_dir "$@" ; ((lines = nfiles > LINES - 1 ? LINES - 1 : nfiles, center = lines / 2)) ; ((qd)) || refresh ; list "${3:-$found}" ; true ; }
get_dir() # dir, search
{
new_dir="${1:-$PWD}" ; [[ -d "$new_dir" ]] || return ; cd "$new_dir" || return ; PWD="$new_dir" ; tabs[$tab]="$PWD" ; [[ "$PWD" == / ]] && sep= || sep=/
[[ "$PPWD" != "$PWD" ]] && { marks["'"]="$n" ; PPWD="$PWD" ; ((gpreview)) || echo "$n" | tee -a $fs/history >> $ghist ; }

cd "$PWD" || return ; inotify_s ; ((etag)) && etag_dir ; shopt -s nocasematch ; geo_prev
files=() ; files_color=() ; mime=() ; nfiles=0 ; search="${2:-$([[ "${dir_file[${tab}_$PPWD]}" ]] || echo "$find_auto")}" ; found= ; s_type=$sort_type0 ; s_reversed=$sort_reversed0
[[ -f .ftlrc_dir ]] && . .ftlrc_dir ; s_type=${sort_type[tab]:-$s_type} ; [[ "${reversed[tab]}" == "-r" ]] && s_reversed=-r || { [[ "${reversed[tab]}" == "0" ]] && s_reversed= ; }

((gpreview)) || nice -15 $ftl_cfg/generators/generator $thumbs &

declare -A uniq_file ; pad=(* ?) ; pad=${#pad[@]} ; pad=${#pad} ; line=0 ; sum=0 ; first_file= ; local LANG=C LC_ALL=C ; dir &
while : ; do read -s -u 4 pnc ; [ $? -gt 128 ] && break ; read -s -u 5 pc ; read -s -u 6 size
	[[ $pnc == $dir_done ]] && break ; ((${uniq_file[$pnc]})) && continue ; uniq_file[$pnc]=1
	((quick_display && nfiles > 0 && 0 == nfiles % quick_display)) && { refresh ; qd=1 ; list $found ; }
	[[ ! -r "$pnc" ]] && pc="\e[7m$pnc\e[m" ; [[ -d "$pnc" && ! -x "$pnc" ]] && pc="\e[31m$pnc"
	[[ "$pnc" =~ '.' ]] && { e=${pnc##*.} ; ((lignore[${e@Q}])) && continue ; } ; pl=${#pnc}
	((etag)) && { etag_tag "$pnc" external_tag external_tag_length ; pc="$external_tag$pc" ; ((pl+=external_tag_length)) ; }
	((show_size)) && { ((sum += size, pl += 5)) ; [[ -d "$pnc" ]] && { ((show_dir_size)) && pc="$(dsize "$pnc") $pc" || pc="     $pc" ; true ; } \
			 || { for u in '' K M G T ; do ((size < 1024)) && printf -v pc "\e[94m%4s\e[m $pc" $size$u && break ; ((size/=1024)) ; done  ; } ; }
	((show_line)) && { ((line++)) ; printf -v pc "\e[2;30m%-${pad}d\e[m¿${pc/\%/%%}" $line ; ((pl += 4)) ; }
	pcl=${pc:0:(( ${#pc} == $pl ? ($COLS - 1) : ( (${#pc} - 4) - $pl ) + ($COLS - 1) )) }
	((pl > (COLS - 1))) && { [[ "$pnc" =~ '.' ]] && e=${pnc##*.} || e= ; pcl=${pcl:0:((${#pcl}-(${#e}+1)))}…${e} ; }
	[[ -f "$pnc" ]] && [[ -z $first_file ]] && first_file=$nfiles
	files_color[$nfiles]="$pcl" ; files[$nfiles]="$PWD$sep$pnc" ; [[ -n "$search" && -z "$found" ]] && [[ "${pnc:0:${#search}}" == "$search" ]] && found=$nfiles ; ((nfiles++))
done

shopt -u nocasematch ; qd=0 ; ((show_size)) && hsum=$(numfmt --to=iec --format ' %4f' "$sum") || hsum=
}

list() # select
{
[[ $1 ]] && dir_file[${tab}_$PWD]=$1 ; file=${dir_file[${tab}_$PWD]:-0}
((file = file > nfiles - 1 ? nfiles - 1 : file)) ; ((nfiles)) && path "${files[file]}" || path_none ; ((gpreview)) || save_state $fs ; selection ; geo_winch
((top = nfiles < lines || file <= center ? 0 : file >= nfiles - center ? nfiles - lines : file - center, bottom = top + lines - 1, bottom = bottom < 0 ? 0 : bottom)) ; l=0

head="${lglyph[lmode[tab]]}${iglyph[imode[tab]]}${pdir_only[tab]}${montage}" ; head=${head:+$head }
((ntabs>1)) && tabsd=' ᵗ'$((tab+1)) || tabsd= ; 
((nfiles)) && { ((qd)) || ((flipi^=1)) ; flip="${flips[$flipi]}" ; } || { head="\e[33m∅ $head$ftag$tabsd" ; header '' "$head" ; tcpreview ; clear_list 0 ; return ; }
((show_stat)) && stat="$(stat -c ' %A %U' "${files[file]}") $(stat -c %s "${files[file]}" | numfmt --to=iec --format '%4f')" || stat= ;
((s_type == 2 && show_date)) && date=$(date -r "${files[file]}" +' %D-%R') || date= ; head="$head$ftag$((file+1))/${nfiles}$hsum$stat$date$tabsd$flip" ; header '' "$head"

for((i=$top ; i <= bottom ; i++))
	do
		cursor=${tags[${files[$i]}]:- } ; [[ $i == $file ]] && cursor="${cursor_color}$cursor\e[m"
		echo -ne "\e[m\e[K$cursor${files_color[i]/¿/$flip}" ; ((i != bottom)) && echo ; ((l++))
	done

((l < LINES - 1)) && echo ; clear_list $i ; ((qd || gpreview)) || { geo_winch ; preview ; }
}

preview() # force_preview
{
((main || prev_all)) || { pane_read ; save_state ; echo "$fs" >$fsp/fs ; echo $my_pane >$fsp/pane ; tmux send -t $main_pane ${C[SIG_REMOTE]} &>/dev/null ; return ; }

((emode)) && { for external in $(externals) ; do $external && { sleep 0.1 ; emode=0 ; return ; } ; done ; } ; emode=0 ;
((${1:-${preview:-$prev_all}})) && { preview= ; for v in $(previewers) ; do $v && { extmode=0 ; return ; } ; done ; extmode=0 ; }
tcpreview
}

try()       { exec 9>&2 2>"$fs/error_log" ; "$@" ; exec 2>&9 ; [[ -s $fs/error_log ]] && { ((gpreview)) && eval "$(error_cmds)" || tmux popup -w 80% "$(error_cmds)" ; } ; }
error_cmds(){ echo "echo -ne '\e[2J\e[H\e[31m' ; cat $fs/error_log | tee -a $fs/errors_log && rm $fs/error_log ; echo '$(cmd_name $REPLY)'" ; }
ewith()     { . ~/.config/ftl/open_with ; R="${C[refresh]}$R" ; true ; }
edir()      { [[ -d "$n" ]] && {  vlc "$n" &>/dev/null & } ; }
ehtml()     { [[ $e =~ html ]] && { ((emode == 1)) && { tcpreview ; w3m -o confirm_qq=0 "$n" ; } || { (qutebrowser "$n" 2>&- &) ; } ; } ; }
eimage()    { [[ $e =~ $ifilter ]] && run_maxed fim -a "$n" "$PWD" ; }
emedia()    { [[ $e =~ $mfilter ]] && { ((emode == 1)) && { mplayer_k ; mplayer -vo null "$n" </dev/null >/dev/null & } || (vlc "$n" &>/dev/null &) ; mplayer=$! ; } ;  }
epdf()      { [[ $e == pdf ]] && { ((emode == 1)) && epdf_vi || { ((emode == 2)) && (mupdf "$n" 2>/dev/null &) && true ; } || { ((emode == 3)) && run_maxed mupdf "$n" ; } ; } ; }
epdf_vi()   { t="$thumbs/pdf/${f}1.txt" ; [[ -e $t ]] || $pgen/pdf "$f" $thumbs/pdf 1 && edit "$t" ; true ; }
etext()     { { [[ $e =~ ^json|yml$ ]] || [[ $mtype =~ ^text ]] ; } && [[ -s "$n" ]] && { tcpreview ; tsplit "$EDITOR ${n@Q}" "33%" '-h -b' -R ; pane_id= ; } ; }
pcbr()      { [[ $e == cbr ]] && { t="$thumbs/cbr/$f.jpg" ; [[ -e $t ]] || $pgen/cbr "$f" "$thumbs/cbr" ; pw3image "$t" ; true ; } ; }
pcbz()      { [[ $e == cbz ]] && { t="$thumbs/cbz/$f.jpg" ; [[ -e $t ]] || $pgen/cbz "$f" "$thumbs/cbz" ; pw3image "$t" ; true ; } ; }
pdir()      { [[ -d "$n" ]] && { ((extmode)) && pdir_tree || { [[ "$montage" ]] && pdir_image || { echo "${ofs:-$fs}" >$fsp/fs ; pdir_dir ; } ; } ; } || { in_pdir= ; pdir_only ; } ; }
pdir_dir()  { ((in_pdir)) && [[ $pane_id ]] && tmux send -t $pane_id ${C[SIG_REFRESH]} || { tmux selectp -t $my_pane ; ctsplit "ftl ${n@Q} $pfs 1" ; in_pdir=1 ; } ; true ; }
pdir_image(){ in_pdir= ; m="$ftl_root/montage/$n/montage.jpg" ; [[ -e "$m" ]] || { header '' "$mh" ; "$pgen/montage" "$ftl_root" "$n" ; header '' "$head" ; } ; pw3image "$m" ; true ; }
pdir_only() { [[ "${pdir_only[tab]}" ]] && { tcpreview ; true ; } || false  ; }
pdir_tree() { ((extmode==1)) && ctsplit "dust -r -f \"$n\" | cat | tail -n +2 ; read -sn1" || ctsplit "ncdu \"$n\"" ; true ; }
phtml()     { [[ $e == html ]] &&  { t="$thumbs/html/$f.txt" ; [[ -e $t ]] || $pgen/html "$f" "$thumbs/html" ; vipreview "$t" ; } ; }
pignore()   { [[ $e ]] && ((pignore["${e@Q}"])) && { mime_get ; in_pdir= ; in_vipreview= ; ptype ; true ; } || false ; }
pimage()    { [[ $e =~ $ifilter ]] && pw3image ; }
plock()     { [[ -e "$fs/lock_preview/$n" ]] && vipreview "$fs/lock_preview/$n" ; }
pmp3()      { [[ $e == mp3 ]] && { pmlive || { ctsplit "/bin/less -R <$(gen_exift) ; read -sn1" ; } ; } ; }
pmp4()      { [[ $e =~ mkv|mp4|flv ]] && { pmlive || { t="$thumbs/$e/$f.jpg" ; [[ -f "$t" ]] || $pgen/$e "$f" "$thumbs/$e"; pw3image "$t" ; true ; } ; } ; }
pmlive()    { ((extmode)) && { mplayer_k ; ctsplit "cat $(gen_exift) ; mplayer -msglevel all=-1 -msglevel statusline=6 -nolirc -msgcolor -novideo -vo null ${n@Q}" ; true ; } ; }
pshell()    { [[ $mtype == 'application/x-shellscript' ]] && vipreview "$n" ; }
ppdf()      { [[ $e == pdf ]] && { ((extmode==2)) && ppdfpng || { t="$thumbs/pdf/$f$extmode.txt" ; [[ -e $t ]] || $pgen/pdf "$f" $thumbs/pdf $extmode && vipreview "$t" ; } ; true ; } ; }
ppdfpng()   { t="$thumbs/$e/et_$(head -c50000 "$n" | md5sum | cut -f1 -d' ').png" ; [[ -f "$t" ]] || { mutool draw -o "$t" "$n" 1 2>/dev/null ; } ; pw3image "$t" ; true ; }
pperl()     { [[ $mtype == application/x-perl ]] && [[ -s "$n" ]] &&  vipreview "$n" ; }
ptext()     { { [[ $e =~ ^json|yml$ ]] || [[ $mtype =~ ^text ]] ; } && [[ -s "$n" ]] && vipreview "$n" ; }
pcomp()     { [[ $e == zip ]] && _pcomp unzip -l || { [[ $e == rar ]] && _pcomp unrar list ; } || { [[ $f =~ \.tar ]] && _pcomp tar --list --verbose -f ; } ; }
_pcomp()    { ((extmode)) && { fp="$fs/$f.txt" ; [[ -e "$fp" ]] || timeout 1 "$@" "$f" >"$fp" ; vipreview "$fp" ; } ; }
ptype()     { ctsplit "echo ${f@Q} ; echo '$mtype' ; file -b ${n@Q} ; stat -c %s ${n@Q} | numfmt --to=iec ; read -sn 100" ; true ; }
pw3image()  { image="${1:-$n}" ; ((in_ftli)) && tmux send -t $pane_id "${image}" C-m || { ctsplit "ftli $pfs \"${image}\"" ; in_ftli=1 ; sleep 0.05 ; tmux selectp -t $my_pane ; } ; }
tcpreview() { [[ "$pane_id" ]] && { tmux killp -t $pane_id &> /dev/null ; in_pdir= ; pane_id= ; in_vipreview= ; in_ftli= ; sleep 0.01 ; } ; }
vipreview() { ((in_vipreview)) && tmux send -t $pane_id ":e ${tail[$1]}$(sed -E 's/\$/\\$/g' <<<"$1")" C-m  || ctsplit "$EDITOR -R ${tail[$1]}${1@Q}" ; in_vipreview=1 ; }
bulkrename(){ bulkedit && bulkverify && { bash $fs/br && tags_clear || read -sn 1 ; } ; true ; }
bulkedit()  { printf "%s\n" "${!tags[@]}" | sed "s/.*/\"&\"/" | tee $fs/bo >$fs/bd ; $EDITOR $fs/bd && { >$fs/br echo 'set -e' ; paste $fs/bo $fs/bd >>$fs/br ; } ; }
bulkverify(){ perl -i -ne '/^([^\t]+)\t([^\t]+)\n/ && { $1 ne $2 && print "mv $1 $2\n" } || print' $fs/br ; $EDITOR $fs/br ; }
cmd_prompt(){ trap 'echo -ne "\e[A\e[K" ; stty -echo ; tput civis' SIGINT ; echo $(rlwrap -p'0;33' -S 'ftl > ' -f $ftl_cfg/cmd_names -H $ftl_root/cmd_history -o cat) ; trap - SIGINT ; }
clear_list(){ for(( i=$1 ; i < LINES - 1  ; i++)) ; do echo -ne "\e[K$flip" ; ((i < LINES - 2)) && echo ; done ; }
ctsplit()   { { in_pdir= ; in_vipreview= ; in_ftli= ; } ; [[ $pane_id ]] && tmux respawnp -k -t $pane_id "$1" &> /dev/null || tsplit "$1" ; }
cmd_name()  { for k in "${!C[@]}" ; do [[ "${C[$k]}" == "$1" ]] && { echo "<$$> command: $k, key: ${shortcuts[$k]}" ; return ; } ; done ; }
cp_mv()     { [[ $1 == ${C[tag_copy]} ]] && cmd="cp -r" || cmd=mv ; tscommand "$cmd $(printf '%q ' "${@:3}") ${2@Q}" ; }
cp_mv_tags(){ declare -A ltags ; tag_get ltags class ; ((${#ltags[@]})) && { cp_mv $1 "$2" "${!ltags[@]}" ; tag_clear $class ; } ; true ; }
dedup()     { [[ -s "$1" ]] && { tac "$1" | awk '!seen[$0]++' | tac | sponge "$1" ; true ; } ; }
delete()    { prompt "delete$1? [y|d|N|c]: " -n1 && [[ $REPLY =~ y|d|c ]] && { [[ $REPLY == c ]] && $RM "$n" || $RM "${@:2}" ; mime=() ; cdir "" "" $file ; }  || { list ; false ; } ; }
delete_tag(){ declare -A ltags ; tag_get ltags class ; ((${#ltags[@]})) && delete " *tags: ${#ltags[@]}* " "${!ltags[@]}" ; [[ $REPLY =~ y|d ]] && tag_clear $class ; }
dir()       { ((lmode[tab]<2)) && files "-type d,l -xtype d" filter2 ; files "-xtype l" filter ; ((lmode[tab]!=1)) && files "-type f,l -xtype f" filter ; dir_done ; }
dir_done()  { echo "$dir_done" >&4 ; echo '' >&5 ; echo 0 >&6 ; }
def_sub()   { for def in "$@" ; do local sub=${def%%:*} ; local body=${def:((${#sub}+1))} ; type "$sub" &>/dev/null || eval "$sub(){ ${body:-:} ; }" ; done ; }
dsize()     { printf "\e[94m%4s\e[m" $(find "$1/" -mindepth 1 -maxdepth 1 ${show_hidden[tab]:+\( ! -iname '.*' \)} -type d,f,l -xtype d,f -printf "1\n" 2>&- | wc -l) ; }
edit()      { tcpreview ; echo -en '\e[?1049h' ; inotify_k ; "${EDITOR}" "${1:-${files[file]}}"; echo -en '\e[?1049h\e[?25h' ; emode=0 ; cdir ; }
files()     { find "$PWD/" -mindepth 1 -maxdepth ${max_depth[tab]:-1} ${show_hidden[tab]:+\( ! -path "*/.*" \)} $1  -printf '%s\t%T@\t%P\n' 2>&- | ftl_filter | $2 ; }
filter()    { rg ${ntfilter[tab]} "${tfilters[tab]}" | rg "${filters[tab]}" | rg "${filters2[tab]}" | { [[ "${rfilters[tab]}" ]] && rg -v "${rfilters[tab]}" || cat ; } | filter2 ; }
filter2()   { ftl_sort | tee >(cat >&4) | lscolors >&5 ; }
filter_rst(){ eval 'ftl_filter(){ cat ; } ; ftl_sort(){ sort_by ; } ; sort_glyph(){ echo ${sglyph[s_type]} ; }' ; }
ftl_env()   { ftl_env=([ftl_pfs]=$pfs [ftl_fs]=$fs) ; for i in "${!ftl_env[@]}" ; do echo -n "${1:--e} $i=${ftl_env[$i]} " ; done ; }
ftl_imode() { ((imode[tab] == 2)) && { tfilters[tab]="$ifilter$" ; ntfilter[tab]='-v' ; } || { ((imode[tab])) && { ntfilter[tab]= ; tfilters[tab]="$ifilter$" ; } || tfilters[tab]= ; } ; }
fzf_go()    { { IFS= read m ; IFS= read p ; } <<<"$1" ; [[ "$p" ]] && { d="$(dirname "$p")" ; [[ -n $m ]] && tab_new "$d" ; cdir "$d" "$(basename "$p")" ; } || list ; }
fzf_rg_go() { { IFS= read m ; IFS= read p ; } <<<"$1" ; [[ "$p" ]] && { g=${p%%:*} && d="$PWD/"$(dirname "$g") ; [[ -n $m ]] && tab_new "$d" ; cdir "$d" "$(basename "$g")" ; } || list ; }
fzf_tag()   { [[ "$2" ]] && while read f ; do [[ "$1" == U ]] && unset -v "tags[$f]" || tags[$PWD/$f]='▪' ; done <<<$2 ; }
gen_exift() { t="$thumbs/$e/$f.et" ; [[ -e "$t" ]] || $ftl_cfg/generators/$e "$f" "$thumbs/$e" ; echo "${t@Q}" ; }
geometry()  { read -r TOP WIDTH LINES COLS LEFT< <(tmux display -p -t $my_pane '#{pane_top} #{window_width} #{pane_height} #{pane_width} #{pane_left}') ; }
geo_prev()  { geometry ; ((${preview:-$prev_all})) && [[ -z $pane_id ]] && ((COLS=(COLS-1) * (100 - ${zooms[zoom]}) / 100)) ; }
geo_winch() { geometry ; WCOLS=$COLS ; WLINES=$LINES ; }
header()    { h="${@:2}$(sort_glyph)$s_reversed$( ((${#tags[@]})) && echo " ${#tags[@]}")$(tsc_head) " ; header_pos "$h" ; echo -e "\e[H\e[K\e[${1:-94}m${PWD:hpl} \e[95m${h:hal}\e[m" ; }
header_pos(){ hal=$((${#1} - ($COLS - 1))) ; hpl=$((${#PWD} + (hal < 0 ? hal : 0) )) ; ((hal = hal < 0 ? 0 : hal, hpl = hpl < 0 ? 0 : hpl)) ; }
inotify_s() { inotify_ & : ; ino1=$! ; ino2=$(ps --ppid $! | grep inotifywait | awk '{print $1}') ; }
inotify_()  { inotifywait --exclude 'index.lock|(.*\.sw.?)' -e create -e delete -e move "$PWD/" 2>&- | { read a b f ; [[ "$f" ]] && tmux send -t "$my_pane" ${C[refresh]} 2>&- ; } ; }
inotify_k() { [[ $ino1 ]] && { kill $ino1 $ino2 2>&- ; ino1= ; ino2= ; } ; }
kbdf()      { [[ $REPLY != $'\e' && $REPLY != "[" && $leader_command != 1 ]] && while read -t 0.01 ; do : ; done ; }
cdfl()      { true 2>&- >&3 && { [[ $REPLY == ${C[quit]} ]] && ((! in_Q)) && printf "%s\n" "${selection[@]}" >&3 || echo >&3 ; } ; }
mime_get()  { [[ "${mime[$n]}" ]] || { mime_cache ; mime[$n]="$(mimemagic "$n")" ; } ;  mtype="${mime[$n]}" ; false ; }
mime_cache(){ while IFS=$': ' read fm mm ; do mime[$PWD/$fm]="$mm" ; done < <(mimemagic "${files[@]:$file:((file + 20))}" 2>&1) ; }
mkapipe()   { for arg in "$@" ; do PIPE=$(mktemp -u) && mkfifo $PIPE && eval "exec $arg<>$PIPE" && rm $PIPE ; done ; }
move()      { ((nf = file + $1, nf = nf < 0 ? 0 : nf >= nfiles ? nfiles - 1 : nf)) ; ((nf != file)) && dir_file[${tab}_$PWD]=$nf ; }
mplayer_k() { ((mplayer)) && { kill $mplayer &>/dev/null ; mplayer= ; } ; }
pane_extra(){ echo "${ofs:-$fs}" >$fsp/fs ; pane_ftl "'$n' $pfs" "$1" "$2" ; sleep 0.05 ; rdir ; tmux selectp -t $new_pane ; echo -e "\e[H\e[K" ; header '2' "$head" ; }
pane_ftl()  { pane_read ; tcpreview ; tsplit "prev_all=0 ftl $1" 30% "$2" $3 ; panes+=($pane_id) ; new_pane=$pane_id ; pane_id= ; printf "%s\n" "${panes[@]}" >$pfs/panes ; }
pane_close(){ pane_read ; ((main && ${#panes[@]})) && { tail -n +2 $pfs/panes | sponge $pfs/panes ; tmux send -t ${panes[0]} ${C[quit]} 2>&- ; sleep 0.03 ; } ; }
pane_next() { pane_read ; pf=0 ; tp=("${panes[@]}" $main_pane "${panes[@]}") ; for p in  "${tp[@]}" ; do [[ $p == $my_pane ]] && pf=1 || { ((pf)) && echo $p && break ; } ; done ; }
pane_read() { <$pfs/pane read main_pane ; [[ -s $pfs/panes ]] && mapfile -t panes < <(grep -w -f <(tmux lsp -F "#{pane_id}") $pfs/panes) ; printf "%s\n" "${panes[@]}" >$pfs/panes ; }
pane_send() { pane_read && for p in "${panes[@]}" ; do tmux send -t $p "$1" &>/dev/null ; done ; }
path()      { n="$1" ; [[ "$n" =~ / ]] && p="${n%/*}" || p= ; [[ ${p:0:1} != "/" ]] && p="$PWD/$p" ; f="${n##*/}" ; b="${f%.*}" ; [[ "$f" =~ '.' ]] && e="${f##*.}" || e= ; }
path_none() { n= ; p= ; f= ; b= ; e= ; }
pdh()       { ((pdhl)) && echo "$$ $my_pane: $1" >>ftl_log ; [[ -f $pfs/pdh ]] && { read pdh <$pfs/pdh ; [[ $pdh ]] && tmux send -t $pdh "$$ $my_pane: ${1//\\n/$'\n'}" ; } ; true ; }
pdh_show()  { [[ -f $pfs/pdh ]] && { read P <$pfs/pdh ; tmux killp -t $P &>/dev/null ; rm $pfs/pdh ; } || { tcpreview ; tsplit "$ftl_cfg/fpdh $pfs" 30% -v -R ; pane_id= ; } ; cdir ; }
pid_2_pane(){ while read -s pi pp ; do [[ $1 == $pp ]] || [[ $(ps -o pid --no-headers --ppid $pp | rg $$) ]] && echo $pi && break ; done < <(tmux lsp -F '#{pane_id} #{pane_pid}') ; }
prev_synch(){ tag_synch ; read ofs <$fsp/fs ; . "$ofs/ftl" ; ftl_imode "${imode[tab]}" ; [[ $1 ]] && { path "$n" ; preview ; } || { cdir "$sdir" '' "$sindex" ; } ; ofs= ; }
prompt()    { exec 2>&9 ; stty echo ; echo -ne '\e[H\e[K\e[33m\e[?25h' ; read -e -rp "$@" ; echo -ne '\e[m' ; stty -echo ; tput civis ; exec 2>"$fs/error_log" ; }
quit()      { tcpreview ; quit2 ; quit_shell ; tmux kill-session -t ftl$$ &>/dev/null ; ((!main)) && { pane_read ; tmux send -t $main_pane ${C[SIG_PANE]} ; } ; rm -rf $fs ; exit 0 ; }
quit2()     { inotify_k ; stty echo ; [[ $pfs == $fs ]] && tbcolor 236 52 ; mplayer_k ; kill $w3iproc &>/dev/null ; refresh "\e[?25h\e[?1049l" ; cdfl ; }
quit_shell(){ [[ $REPLY != ${A[q]} ]] && [[ $shell_id ]] && tmux killp -t $shell_id &> /dev/null ; }
rdir()      { get_dir "$1" ; ((lines = nfiles > LINES - 1 ? LINES - 1 : nfiles, center = lines / 2)) ; qd=${2:-1} ; list ; qd=0 ; }
refresh()   { echo -ne "\e[?25l$1" ; } 
run_maxed() { run_maxed=1 ; ((run_maxed)) && { aw=$(xdotool getwindowfocus -f) ; xdotool windowminimize $aw ; } ; "$@" 2>/dev/null ; ((run_maxed)) && { wmctrl -ia $aw ; } ; true ; }
selection() { selection=() ; ((${#tags[@]})) && selection+=("${!tags[@]}") || { ((nfiles)) && selection=("${files[file]}") ; } ; }
shell_pane(){ tcpreview ; P=$pane_id; tsplit bash 30% -v -U ; shell_id=$pane_id ; pane_id=$P; sleep 0.3 ; ((shell_file)) && shell_send "$(printf "%s " "${selection[@]@Q}")" C-b ; cdir ; }
shell_send(){ { [[ $shell_id ]] && $(tmux has -t $shell_id 2>&-) ; } && tmux send -t $shell_id "$@" ; }
shell_pop() { tmux popup -E -d "$PWD" -w 80%  ; }
sort_by()   { sort $s_reversed ${sort_filters[s_type]} | tee >(cut -f 1 >&6) | cut -f 3- ; }
sort_glyph(){ echo ${sglyph[s_type]} ; }
save_state(){ declare -p tags | sed 's/\-A/-A -g/' >$fs/tags ; echo "$stagsi" >$fsp/stagsi ; echo $fs >$fsp/fs
		((!nfiles)) && : >${1:-$fs}/ftl || >${1:-$fs}/ftl echo "\
			sdir=\"${files[file]}\"	; sindex=${dir_file[${tab}_${files[file]}]}	; n=\"$n\"
			imode[tab]=${imode[tab]}; xxx=0						; sort_type[tab]=${sort_type[tab]}
			ftag=$ftag		; filters[tab]=\"${filters[tab]}\"		; filters2[tab]=\"${filters2[tab]}\" 
			etag=$etag		; show_size=$show_size				; show_dir_size=$show_dir_size" ; }
tab_close() { (($ntabs > 1)) && { tabs[$tab]= ; ((ntabs--)) ; tab_next ; cdir ${tabs[tab]} ; } ; }
tab_new()   { dir="$1" ; [[ "$dir" == '.' ]] && dir= ; [[ "$dir" =~ ^/ ]] || dir="$PWD/$dir" ; tabs+=("$dir") && ((tab=${#tabs[@]} - 1, ntabs++)) ; rfilters[tab]=$rfilter0 ; }
tab_next()  { ((tab++)) ; for i in "${tabs[@]:$tab}" TAB_RESET "${tabs[@]}" ; do [[ "$i" == TAB_RESET ]] && tab=0 && continue ; [[ -n "$i" ]] && break ; ((tab++)) ; done ; true ; }
tag_check() { for tag in "${!tags[@]}" ; do [[ -e "$tag" ]] || unset -v "tags[$tag]" ; done ; ((${#tags[@]} != 0)) ; }
tag_class() { tag_ntags ; ((${#ntags[@]}>1)) && { echo "$(printf "%s\n" "${!ntags[@]}" | sort | fzf-tmux -p 80% --cycle --reverse --info=inline )" ; } || echo "${tags[$t]}" ; }
tags_clear(){ ((stagsi++)) ; tags=() ; true ; }
tag_ntags() { ((stagsi++)) ; ntags=() ; for t in "${!tags[@]}" ; do ntags[${tags[$t]}]=1 ; done ; }
tag_clear() { for t in "${!tags[@]}" ; do [[ "$1" == "${tags[$t]}" ]] && { unset -v "tags[$t]" ; ((stagsi++)) ; } ; done ; }
tag_flip()  { ((stagsi++)) ; [[ ${tags[$1]} ]] && unset -v "tags[$1]" || tags[$1]=${2:-▪} ; }
tag_get()   { ((${#tags[@]})) && { declare -n rtags="$1" rclass="$2" ; rclass=$(tag_class) ; list ; for t in "${!tags[@]}" ; do [[ $rclass == ${tags[$t]} ]] && rtags[$t]=1 ; done ; } ; }
tag_synch() { 2>&- read ostagsi <$fsp/stagsi ; ((auto_tags && ostagsi != stagsi)) && stagsi=$ostagsi && read ofs <$fsp/fs && source $ofs/tags && rdir ; ofs= ; false ; }
tbcolor()   { tmux set pane-border-style "fg=color$1" ; tmux set pane-active-border-style "fg=color$2" ; sleep 0.01 ; }
tscommand() { tmux new -A -d -s ftl$$ ; tmux neww -t ftl$$ -d ${@:2} "echo -e \"\e[33m$(date)\nftl> $1\e[m\" ; { $1 ; } || { echo -e \"\e[7;31mftl: failed\e[m\\n\" ; exec bash ; }" ; }
tsc_head()  { w=$(tmux lsw -t ftl$$ 2>&- | wc -l) ; ((w > 1)) && echo -e " \e[33m!\e[0m" ; }
tsplit()    { tmux sp $(ftl_env) $5 -t $my_pane ${3:--h} -l ${2:-${zooms[zoom]}%} -c "$PWD" "$1" && { sleep 0.03 ; pane_id=$(tmux display -p '#{pane_id}') && tselectp $4 ; } ; }
tselectp()  { tmux selectp -t $pane_id ${1:--L} ; }
winch()     { geometry ; { ((!in_ftli)) && [[ "$WCOLS" != "$COLS" ]] || [[ "$WLINES" != "$LINES" ]] ; } && ((winch)) && R="${C[refresh]}$R" ; }
zoom()      { geometry ; [[ $pane_id ]] && read -r COLS_P < <(tmux display -p -t $pane_id '#{pane_width}') || COLS_P=0 ; ((x = ( ($COLS + $COLS_P) * ${zooms[$zoom]} ) / 100)) ; }

def_sub 'ftl_sort:sort_by' 'ftl_filter:cat' etag_dir etag_tag 'ext_bindings:false' externals previewers ; . ~/.config/ftl/ftl.eb ; [[ $TMUX ]] && ftl "$@" || echo 'ftl: run in tmux'
