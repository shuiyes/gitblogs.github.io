# ubuntu ～/.bashrc 常用修改

```
#shuiye
# terminal display name
PS1="\[\e]0;「\t>」\w\a\]\u:\w$ "

# cd ..
function cd3(){
cd ../../../
}
function cd4(){
cd ../../../../
}
function cd5(){
cd ../../../../../
}
function cd6(){
cd ../../../../../../
}

# cgrep jgrep
# alias jgrep="find ./ -name '*.java' | xargs grep -in '$1'"
function jgrep() {
    find ./ -name "*.java" | xargs grep -in "$1"
}

function cgrep() {
    find ./ -name "*.c*" | xargs grep -in "$1"
}

function hgrep() {
    find ./ -name "*.h" | xargs grep -in "$1"
}


function xgrep() {
    find ./ -name "*.xml" | xargs grep -in "$1"
}

function mkgrep() {
    find ./ -name "*.mk" | xargs grep -in "$1"
}

function loggrep() {
    find ./ -name "*_log" | xargs grep -in "$1"
}

function ff() {
    find ./ -name "$1"
}

function adblogcat() {
    adb logcat -b main -v time | grep -in "$1"
}

function slog() {
    adb logcat -b main -v time | grep -n "SHUIYES"
}

function mmi() {
    rm -rf gen
    mm
    if [ $? -eq 0 ]
    then
        echo "编译成功"
        ./install.sh
    else
        echo "编译失败"
    fi
}

alias pss="adb shell ps | grep -in shuiyes"
function kls() {
    adb shell kill -9 "$1"
}
alias lgs="adb logcat | grep -in shuiyes"

alias mkfls="make -j8 flashfiles BUILD_OSAS=1"

alias lgc="adb logcat -c"

alias adbrt="adb root"

alias adbrm="adb remount"

alias adbrb="adb reboot"

alias adbrs="adb shell stop;adb shell start"

alias adbback="adb shell input keyevent 4"

alias sb="source build/envsetup.sh"

alias lunch1="choosecombo release full_rlk8321_tb_rc_m eng develop"
alias lunch2="lunch sf3gr_telit_he922-eng"

#alias pushcode="git push origin HEAD:refs/for/dev_orange_archermind"
alias pushorange="git push origin HEAD:refs/for/dev_orange"
alias pushsystech="git push origin HEAD:refs/for/systech"

alias opendir="nautilus"

alias jdgui="/home/archermind/shuiye/apktool/jd-gui/jd-gui"

alias fuck='eval $(thefuck $(fc -ln -1)); history -r'

# JDK config
#export JAVA_HOME=/usr/lib/jvm/jdk1.6.0_45
#export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_45
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

# adb config
export PATH=/home/archermind/shuiye/adt-bundle-linux-x86_64-20140702/sdk/platform-tools:$PATH
export PATH=/home/archermind/shuiye/adt-bundle-linux-x86_64-20140702/sdk/tools:$PATH

# intel icc
export ICC_ROOT_PATH=/work/icc/
export ICC_PATH=$ICC_ROOT_PATH/bin/ia32
export ICC_LD_LIBRARY_PATH=$ICC_ROOT_PATH/compiler/lib/ia32
export ICC_LIBRARY_PATH=$ICC_ROOT_PATH/compiler/lib/ia32
export INTEL_LICENSE_FILE=$ICC_ROOT_PATH/licenses
export PATH=$ICC_PATH:$PATH
export LD_LIBRARY_PATH=$ICC_LD_LIBRARY_PATH:$LD_LIBRARY_PATH
export LIBRARY_PATH=$ICC_LIBRARY_PATH:$LIBRARY_PATH
export INTEL_LICENSE_FILE=$INTEL_LICENSE_FILE:$INTEL_LICENSE_FILE

```
