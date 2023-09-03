# Joshuto
拷贝自theniceboy。
## 配置
将本库拉取到`.config`下即可，名称为`joshuto`，此外，需要赋予preview_file.sh可执行权限，否则无法使用该文件定义的功能，如文件预览
### 图片预览
还需要安装`exiftool`,配合获取图片的信息，然后会利用尺寸去展示图片。
这里对官方的图片展示脚本进行了简化，并将展示范围往下移动了一点，从而去展示图片的尺度信息, 该操作来自于B站的一个UP主。
####  额外的配置
为了能够预览图片还需要在/usr/bin/下重新配置joshuto的启动形式，脚本如下
```bash
#!/usr/bin/env bash

if [ -n "$DISPLAY" ] && command -v ueberzug > /dev/null; then
    export joshuto_wrap_id="$$"
    export joshuto_wrap_tmp="$(mktemp -d -t joshuto-wrap-$joshuto_wrap_id-XXXXXX)"
    export joshuto_wrap_ueber_fifo="$joshuto_wrap_tmp/fifo"
    export joshuto_wrap_pid_file="$joshuto_wrap_tmp/pid"
    export joshuto_wrap_preview_meta="$joshuto_wrap_tmp/preview-meta"
    export joshuto_wrap_ueber_identifier="preview"

    function start_ueberzug {
        mkfifo "${joshuto_wrap_ueber_fifo}"
        tail --follow "$joshuto_wrap_ueber_fifo" | ueberzug layer  --parser bash &
        echo "$!" > "$joshuto_wrap_pid_file"
        mkdir -p "$joshuto_wrap_preview_meta"
    }

    function stop_ueberzug {
        ueberzug_pid=`cat "$joshuto_wrap_pid_file"`
        kill "$ueberzug_pid"
        rm -rf "$joshuto_wrap_tmp"
    }

    function show_image {
        >"${joshuto_wrap_ueber_fifo}" declare -A -p cmd=( \
                [action]=add [identifier]="${joshuto_wrap_ueber_identifier}" \
                [x]="${2}" [y]="${3}" \
                [width]="${4}" [height]="${5}" \
                [path]="${1}")
    }

    function remove_image {
        >"${joshuto_wrap_ueber_fifo}" declare -A -p cmd=( \
            [action]=remove [identifier]="${joshuto_wrap_ueber_identifier}")
    }

    function get_preview_meta_file {
        echo "$joshuto_wrap_preview_meta/$(echo "$1" | md5sum | sed 's/ //g')"
    }

    export -f get_preview_meta_file
    export -f show_image
    export -f remove_image

    trap stop_ueberzug EXIT QUIT INT TERM
    start_ueberzug
    echo "ueberzug started"
fi

joshuto "$@"
exit $?
```
