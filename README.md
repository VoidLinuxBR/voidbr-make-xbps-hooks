# xbps-hooks

Impiementa hooks like Pacman no xbps

# Como aplicar o patch de hooks no xbps e compilar

Passo a passo completo, do zero, com o patch mais recente
(`0001-xbps-hooks-support.patch`, já com a correção do deadlock do
pkgdb).

## 1. Clonar o repositório limpo

```sh
git clone https://github.com/void-linux/xbps
cd xbps
```

Se essa pasta já tiver algum patch antigo aplicado, limpe antes:

```sh
git checkout -- .
git clean -fd
```

Copie o arquivo `0001-xbps-hooks-support.patch` pra dentro dessa
pasta `xbps/`.

## 2. Validar e aplicar o patch

```sh
# Validar que aplica limpo (não muda nada ainda)
git apply --check 0001-xbps-hooks-support.patch && echo "OK, aplica limpo"

# Aplicar de verdade
git apply 0001-xbps-hooks-support.patch
```

## 3. Instalar dependências de build

```sh
sudo xbps-install -Sy libarchive-devel openssl-devel zlib-devel
```

## 4. Configurar e compilar

```sh
./configure --enable-rpath --prefix=/usr --sysconfdir=/etc
make -j$(nproc)
```

## 5. Instalar num diretório separado (sem tocar no sistema)

```sh
make DESTDIR=~/xbps-hook install clean

export PATH=~/xbps-hook/usr/bin:$PATH
export LD_LIBRARY_PATH=~/xbps-hook/usr/lib
```

Confirmar que é o binário certo:

```sh
xbps-install --version
```

## 6. Testar

Como o `PATH`/`LD_LIBRARY_PATH` só valem nessa sessão de shell, e o
`sudo`/`doas` reseta variáveis de ambiente por padrão, use `sudo env`
(ou `doas env`) explicitamente:

```sh
sudo env PATH="$PATH" LD_LIBRARY_PATH="$LD_LIBRARY_PATH" xbps-install -f yasm
```

Deve terminar normalmente, sem ficar preso em
`WARNING: package database locked, waiting...`.

## Onde ficam os hooks

```
/etc/xbps.d/hooks.d/*.hook
```

Cada arquivo `.hook` usa o formato `[Trigger]`/`[Action]`
(`Operation`, `Type`, `Target`, `When`, `Exec`, `NeedsTargets`),
igual ao `alpm-hooks(5)` do pacman.
