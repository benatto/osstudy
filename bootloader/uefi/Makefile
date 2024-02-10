all: main.o uefi.img
	parted uefi.img -s -a minimal mklabel gpt
	parted uefi.img -s -a minimal mkpart EFI FAT16 2048s 93716s
	parted uefi.img -s -a minimal toggle 1 boot
	dd if=/dev/zero of=/tmp/part.img bs=512 count=91669
	mformat -i /tmp/part.img -h 32 -t 32 -n 64 -c 1
	mcopy -i /tmp/part.img main.efi ::
	dd if=/tmp/part.img of=uefi.img bs=512 count=91669 seek=2048 conv=notrunc
	rm /tmp/part.img

main.o: main.c
	gcc -Ignu-efi/inc -fpic -ffreestanding -fno-stack-protector -fno-stack-check -fshort-wchar \
	-ggdb -mno-red-zone -maccumulate-outgoing-args -c main.c -o main.o

uefi.img: main.o
	ld -shared -Bsymbolic -Lgnu-efi/x86_64/lib -Lgnu-efi/x86_64/gnuefi -Tgnu-efi/gnuefi/elf_x86_64_efi.lds \
	gnu-efi/x86_64/gnuefi/crt0-efi-x86_64.o main.o -o main.so -lgnuefi -lefi
	objcopy --only-keep-debug main.so main.debug
	objcopy -j .text -j .sdata -j .data -j .dynamic -j .dynsym  -j .rel -j .rela -j .rel.* \
	 -j .rela.* -j .reloc --target efi-app-x86_64 --subsystem=10 main.so main.efi
	dd if=/dev/zero of=uefi.img bs=512 count=93750


clean:
	rm main.o main.so main.efi uefi.img

run:
	sudo qemu-system-x86_64 -s -S -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE.fd,readonly=on -drive if=pflash,format=raw,unit=1,file=/usr/share/OVMF/OVMF_VARS.fd -net none -drive file=uefi.img,format=raw,if=ide -debugcon file:debug.log -global isa-debugcon.iobase=0x402
