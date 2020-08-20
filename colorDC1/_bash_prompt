#!/usr/bin/env bash

function get_username() {
    if [[ -z "$IPLANT_USER" ]]; then
        whoami
    else
        echo "$IPLANT_USER"
    fi
}

function get_hostname() {
    if [[ -z "$REDIRECT_URL" ]]; then
        hostname
    else
        echo "$REDIRECT_URL" | cut -d "/" -f 3
    fi
}

export PS1='\e[01;32m$(get_username)@$(get_hostname)\e[m$ '
