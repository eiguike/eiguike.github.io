---
layout: post
author: eiguike
title:  "computador pessoal com GPU passthrough"
date:   2024-07-27 00:14:01 -0300
categories: gpu passthrough nvidia games vfio
---

olá!

faz um tempo que não posto nada e pretendo continuar assim, hehee.

porém recentemente descobri como fazer um `GPU passthrough` no computador pessoal que utilizo tanto para jogos como no trabalho.

o computador é um desktop e é bem parrudo, segue as configurações:
- AMD Ryzen 7800x3D
- NVIDIA Geforce 4090
- 64 GB DDR5
- AsRock x670e Steel Legend

apesar do processador não ser tão recomendável para realizar isso, a GPU com toda certeza é, e eu não consigo utilizar de maneira 100% a capacidade dela dentro de um Linux. até depois de conseguir fazer essa façanha.

### mas o que diabos é GPU passthrough?
R: é basicamente uma forma de você conseguir utilizar a GPU física como se estivesse instalado em uma máquina virtual, sem qualquer interface que possa deixa-lá mais lenta.

### e por quê gostaria de utilizar essa técnica?
R: caso queira utilizar Linux e o Windows ao mesmo tempo sem perder a performance da GPU para jogos

### quais são os requisitos?
1. aqui é essencial verificar se o seu processador é compatível com virtualização, acredito os últimos processaores da AMD/Intel já sejam compatíveis com isso;
2. ter uma outra placa de vídeo (no meu setup o processor Ryzen 7800x3D ele já tem uma placa de vídeo discreta, logo eu consigo utiliza-lá ela para o Linux e a GPU dedicada para o Windows);
3. placa mãe deve ser compatível com as features de `immo`, não faço muita ideia do que seja isso;

### como fazer?
aqui assumirei que você fará isso em um Linux para hostear uma máquina virtual Windows e já tenha o QEMU + Virtual Machine Manager instalado.

1. extraia os IDs da sua GPU, via comando `lspci --n | grep NVIDIA`:
![image](https://github.com/user-attachments/assets/3c7ee86e-5650-4586-99c5-bc5ca860c2a0)

2. modifique o arquivo `/etc/default/grub` e modifique a seguinte linha:
![image](https://github.com/user-attachments/assets/5a1f3fb8-9219-42ea-bbf9-81921c8d2c13)

a linha terá o comando `iommu=pt vfio-pci.ids=<gpu_id>,<gpu_id>` (se você estiver usando um processador Intel, adicione o comando `intel_iommu=on`)

3. após salvar o arquivo, atualize o grub com esse comand:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
4. reinicie a máquina
  
5. altere o seguinte arquivo `sudo vim /etc/modprobe.d/vfio.conf` e adicione as seguintes linhas:
```
blacklist nouveau
blacklist snd_hda_intel
options vfio-pci ids=<gpu_id>,<gpu_id>
softdep drm pre: vfio-pci
```
6. atualize o kernel com esse seguinte comando: `sudo update-initramfs -c -k $(uname -r)` e não deve retornar erros;
  
7. reinicie a máquina.
   
8. para verificar se deu tudo certo, a saída desse comando `lspci -k | grep -E "vfio-pci|NVIDIA"` terá que retornar `vfio-pci`, caso contrário, tente remover os driver proprietários instalados da NVIDIA.

9. inicie a máquina e adicione o hardware no QEMU.

10. instale o Windows e só jogar! 

### possíveis problemas
infelizmente nem tudo são flores. utilizando este metódo você até consegue jogar de forma perfomática, porém deve-se tomar o cuidado com jogos multiplayer, onde que o anticheat pode te banir de forma permanente.

apesar da máquina ser um Windows, ainda há diversas formas que anticheat conseguem detecter que é uma máquina virtual. eu acabei não indo muito a fundo e acabei me desanimando um pouco, porém para quem quiser procurar mais artigos só utilizar a palavra `virtual machine spoofing`.

é isso galera, obrigado por ler até aqui!

### referências
[How To USE QEMU KVM GPU Passthrough in Linux Using VFIO](https://www.youtube.com/watch?v=g--fe8_kEcw)

[How To install QEMU KVM & VirtManager on Ubuntu](https://www.youtube.com/watch?v=4m6eHhPypWI)

[Enabling 64-bit PCI Addressing in the Switching the TianoCore UEFI BIOS](https://wiki.gentoo.org/wiki/GPU_passthrough_with_libvirt_qemu_kvm#Enabling_64-bit_PCI_Addressing_in_the_Switching_the_TianoCore_UEFI_BIOS)
