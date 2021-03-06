## Домашнее задание: [Unix Command Line](https://github.com/yandex-shri/lectures/blob/master/04-unix-cli.md)

Написать сценарий, который находит все файлы не входящие в SVN/Git и перемещает их в ~/.Trash/.

    git ls-files . --exclude-standard --others -z | xargs -0 -I {} mv {} ~/.Trash/

присылайте пулл реквесты с решением для SVN или с более элегантным подходом.

См. также: [пост про домашние задания](http://clubs.ya.ru/4611686018427468886/replies.xml?item_no=450).

#!/bin/sh

usage() {
cat << EOF
Usage: cleanup.sh [OPTIONS] [ARGUMENTS]
  Clean up untracked files from repository.

OPTIONS:
  -h      print help message
  -g      clean up git repository
  -s      clean up svn repository

Examples:
  cleanup.sh -h
  cleanup.sh -g 
  cleanup.sh -g some/repository/
  cleanup.sh -s some/repository/

EOF
}

while getopts “hsg” OPTION
do
	case $OPTION in
		h)
			usage
			exit 1
			shift;;
		s)
			SVN=1
			shift;;
		g)
			GIT=1
			shift;;	
	esac
done

if [ "$1" != "" ] 
then
	PATH2REP=`readlink -e $1`
	cd $PATH2REP
fi

if [ $GIT ]
then
	# didn't find simple way to save output that contains \0 into variable correctly
	# so decide to use base64:
	FILELIST=`git ls-files -oz --exclude-standard 2>/dev/null | base64`
	echo $FILELIST | base64 -d | xargs -0 -I {} cp --parent {} ~/.Trash/
	echo $FILELIST | base64 -d | xargs -0 -I {} rm {}

	# another way. Not shure if it is faster to call "git ls-files" twice
	# git ls-files -oz --exclude-standard 2>/dev/null | xargs -0 -I {} cp --parent {} ~/.Trash/
	# git ls-files -oz --exclude-standard 2>/dev/null | xargs -0 -I {} rm {}
elif [ $SVN ]
then 
	FILELIST=`svn st 2>/dev/null | grep ? | awk '{print $2}'`
	echo $FILELIST | xargs -I {} cp --parent {} ~/.Trash/
	echo $FILELIST | xargs -I {} rm {}

	# again as previously with git, another way. Here is no encoding/decoding, because there is no option in "svn st" like git's "-z"
	# but two calls "grep" and "awk", so again not shure if it is faster:
	# svn st 2>/dev/null | grep ? | awk '{print $2}' | xargs -I {} cp --parent {} ~/.Trash/
	# svn st 2>/dev/null | grep ? | awk '{print $2}' | xargs -I {} rm {}
else
	echo "Missed repository type parameter"
fi

if [ "$FILELIST" = '' ]
then 
	echo "No files to move"
fi

exit