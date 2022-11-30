
1. Создаем переменную
      wallet=ТУТ Кошелек
      echo 'export wallet='$wallet >> $HOME/.bash_profile


2. выполняем команды 

        #!/bin/bash
        exists()
        {
          command -v "$1" >/dev/null 2>&1
        }
        if exists curl; then
        echo ''
        else
          sudo apt update && sudo apt install curl -y < "/dev/null"
        fi
        bash_profile=$HOME/.bash_profile
        if [ -f "$bash_profile" ]; then
            . $HOME/.bash_profile
        fi
        sleep 1 && curl -s https://api.nodes.guru/logo.sh | bash && sleep 1

        if [ ! $node_id ]; then
        read -p "Enter node name: " node_id
        echo 'export node_id='\"${node_id}\" >> $HOME/.bash_profile
        fi
        echo 'source $HOME/.bashrc' >> $HOME/.bash_profile
        . $HOME/.bash_profile
        echo 'Your node name: ' $node_id

        sudo apt update < "/dev/null"

        sleep 1

        sudo dpkg --configure -a
        sudo apt install ufw make clang pkg-config libssl-dev build-essential git -y -qq < "/dev/null"
        sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
        source $HOME/.cargo/env
        cd $HOME
        rm -rf nym
        git clone https://github.com/nymtech/nym.git
        cd nym
        git reset --hard
        git pull
        git checkout nym-binaries-1.1.0
        cargo build -p nym-mixnode --release
        #cargo build --release
        sudo mv target/release/nym-mixnode /usr/local/bin/
        nym-mixnode init --id $node_id --host $(curl ifconfig.me) --wallet-address $wallet
        sudo ufw allow 1789,1790,8000,22,80,443/tcp
        sed -i.def "s|validator_api_urls.*|validator_api_urls = [\
                'https://validator.nymtech.net/api/'|" $HOME/.nym/mixnodes/*/config/config.toml
        sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
        Storage=persistent
        EOF
        sudo systemctl restart systemd-journald
        sudo tee <<EOF >/dev/null /etc/systemd/system/nym-mixnode.service
        [Unit]
        Description=Nym Mixnode

        [Service]
        User=$USER
        ExecStart=/usr/local/bin/nym-mixnode run --id '$node_id'
        KillSignal=SIGINT
        Restart=on-failure
        RestartSec=30
        StartLimitInterval=350
        StartLimitBurst=10
        WantedBy=multi-user.target
        EOF
        echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf
        sudo systemctl daemon-reload
        sudo systemctl enable nym-mixnode
        sudo systemctl restart nym-mixnode
        echo -e '\n\e[42mCheck node status\e[0m\n' && sleep 1
        if [[ `service nym-mixnode status | grep active` =~ "running" ]]; then
          echo -e "Your Nym node \e[32minstalled and works\e[39m!"
          echo -e "You can check node status by the command \e[7mservice nym-mixnode status\e[0m"
          echo -e "Press \e[7mQ\e[0m for exit from status menu"
        else
          echo -e "Your Nym node \e[31mwas not installed correctly\e[39m, please reinstall."
        fi

3. 
берем данные для заполнения в  приложение, открываем приложение заполняем бондинг
    . $HOME/.bash_profile
    nym-mixnode node-details --id $node_id


4. Смотрим микс
  journalctl -u nym-mixnode -o cat -f | grep "Since startup mixed"



5. Дополнительно (инфа на нодгуру)

    Полезные команды
    Добавить описание для вашей миксноды(имя которое будет отображаться в эксплорере):

    nym-mixnode describe --id $node_id
    systemctl restart nym-mixnode
    Посмотреть сколько пакетов смиксила ваша нода:

    journalctl -u nym-mixnode -o cat -f | grep "Since startup mixed"
    Перезапустить ноду:

    systemctl restart nym-mixnode
    Обновление ноды

    wget -O nym_update.sh https://api.nodes.guru/nym_update.sh && chmod +x nym_update.sh && ./nym_update.sh
    Расширить ulimit (важно для работы ноды в будущем):

    – вариант в одну строчку

    wget -O nym_ulimit.sh https://api.nodes.guru/nym_ulimit.sh && chmod +x nym_ulimit.sh && ./nym_ulimit.sh
    – вариант вручную

    echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf
    sudo systemctl daemon-reload
    sudo systemctl stop nym-mixnode
    sudo systemctl start nym-mixnode
    проверяем, что значение ulimit=65535

    grep -i "Max open files" /proc/$(ps -A -o pid,cmd|grep nym-mixnode | grep -v grep |head -n 1 | awk '{print $1}')/limits
    Проверка ipv6
    Посмотреть свой внешний ipv6:

    curl http://v4v6.ipv6-test.com/api/myip.php && echo
    Проверить связь с google.de по ipv6:

    ping6 www.google.de
    Посмотреть есть ли ipv6 в сетевых настройках сервера:

    hostname -I
    Расшифровка ваших public ключей:
    Выполняем команды (snapd на вашей OS может устанавливаться иначе):

    apt install snapd
    snap install base58
    ls -1 $HOME/.nym/mixnodes/*/data/public_identity.pem | while read F; do echo === $F ===; grep -v ^- $F | openssl base64 -A -d | base58; echo; done
    ls -1 $HOME/.nym/mixnodes/
