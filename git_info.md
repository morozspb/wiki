git clone git@github.com:morozspb/wiki.git

1. git add . ; git add -A
2. git commit -m "Краткое описание изменений"
3. git push -u origin master ; git push \
origin - означает репозиторий на компьютере, то есть ваш форк. Слово origin — часть команды, не меняйте это название на своё.

### Настройка
git config --global user.name "ваше имя"\
git config --global user.email email@example.com

ssh-keygen -t ed25519 -C "your_email@example.com" - генерация ключей(C:\Users\USERPROFILE\\.ssh)\
.pub - добавляется на https://github.com/settings/keys \
 ssh -T git@github.com -проверить соединение SSH

###
git config --list
git config --edit 