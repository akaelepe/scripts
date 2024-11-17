Jose te dejo el bloque de código que he usado para el script de la tarea 9.
## EJERCICIO 1  
~~~
#!/bin/bash

#~ SCRIPT DE ASO TAREA9 KVM - ALEJANDRO LAMPREA PÉREZ
#~ En este script se crea un menú para gestionar/automatizar las configuraciones de red en KVM.
#~ Se podrá consultar el estado, ver la configuración de red, activarla o desactivarla, iniciarla y modificarla.

while true; do
	clear
	echo "MENÚ DE GESTIÓN DE LAS REDES KVM"
	echo "--------------------------------"
	echo "1) Consultar el estado de la red"
	echo "2) Ver la configuración de la red"
	echo "3) Activar/Desactivar la red"
	echo "4) Inicializar/No inicializar la red"
	echo "5) Modificar la configuración de red"
	echo "6) Salir"
	read -p "Elige una opción: " menu
	
	case $menu in
		1)
			echo "Estado de la red:"
			virsh net-list --all | grep -w "default"
			;;
		2)
			echo "Configuración de la red Default:"
			virsh net-dumpxml default
			;;
		3)
			echo "Activar o desactivar la red..."
			sleep 3
			read -p "Para activarla pulsa '1', para desactivarla '0' " trigger
			if [ "$trigger" -eq 1 ]; then
				virsh net-start default
				echo "Configurando..."
				sleep 5
				echo "La red ha sido activada."
			elif [ "$trigger" -eq 0 ]; then
				virsh net-destroy default
				echo "Configurando..."
				sleep 5
				echo "La red ha sido desactivada."
			else
				echo "Opción inválida, debes elegir '1' o '0'"
			fi
			;;
		4)
			echo "Inicializar/No inicializar la red..."
			sleep 3
			read -p "Para inicializar la red usa '1', para no inicializarla '0'." trigger2
			if [ "$trigger2" -eq 1 ]; then 
				virsh net-autostart default
				echo "Procesando inicialización..."
				sleep 5
				echo "La red se iniciará automáticamente al arrancar."
			elif [ "$trigger2" -eq 0 ]; then
				virsh net-autostart --disable default
				echo "Procesando la NO inicialización..."
				sleep 5
				echo "La red no se iniciará automáticamente al arrancar."
			else 
				echo "Opción inválida, debes elegir '1' o '0'."
			fi
			;;
		5)
			echo "Modificar la configuración de la red."
			read -p "Introduzca el nombre del archivo XML para la nueva configuración: " modify
			if [ -f "$modify" ]; then
				echo "Aplicando cambios en la configuración desde el archivo '$modify'..."
				sleep 3
				virsh net-define "$modify"
				echo "Reiniciando la red para los nuevos cambios..."
				sleep 3
				virsh net-destroy default
				virsh net-start default
				echo "Listo, la nueva configuración de la red está lista."
			else
				echo "El archivo introducido no existe."
			fi
			;;
		6)
			echo "Saliendo del menú..."
			sleep 3
			exit 0
			;;
		*)
			echo "Ha introducido un número inválido. Pruebe de nuevo."
			;;
	esac
	read -p "Pulse 'Enter' para continuar."
done
~~~			

## EJERCICIO 2  
~~~
#!/bin/bash
#~ EJERCICIO 2 DE LA TAREA 9 - ALEJANDRO LAMPREA PÉREZ


crear_mv() {
    while true; do
        echo "Crear máquina virtual (Debian)"
        read -p "Nombre de la máquina virtual: " nombre_mv
        read -p "Cantidad de RAM (en MB): " ram
        read -p "Tamaño del disco (en GB): " disco
        read -p "Número de CPUs: " cpu
        read -p "Red (puente/nat): " red

        ruta_iso="/home/aleserver/Descargas/debian-12.8.0-amd64-netinst.iso"

        if [[ ! -f "$ruta_iso" ]]; then
            echo "Error: No se encontró la ISO de Debian en $ruta_iso"
            echo "Asegúrate de que la ISO de Debian esté en la ubicación correcta."
            return
        fi

        echo "Creando la máquina virtual $nombre_mv..."
        virt-install \
            --name "$nombre_mv" \
            --ram "$ram" \
            --vcpus "$cpu" \
            --disk size="$disco",format=qcow2 \
            --os-type linux \
            --os-variant debian11 \
            --network "$red" \
            --cdrom "$ruta_iso" \
            --graphics vnc --noautoconsole

        echo "Máquina virtual creada con éxito."
        read -p "¿Quieres crear otra máquina virtual? (s/n): " eleccion
        if [[ $eleccion != "s" ]]; then
            break
        fi
    done
}

borrar_mv() {
    while true; do
        echo "Máquinas virtuales existentes:"
        virsh list --all
        read -p "Introduce el nombre de la máquina virtual que quieres eliminar: " nombre_mv

        if virsh list --all | grep -q "$nombre_mv"; then
            virsh destroy "$nombre_mv" &>/dev/null
            virsh undefine "$nombre_mv"
            echo "Máquina virtual $nombre_mv eliminada con éxito."
        else
            echo "La máquina virtual $nombre_mv no existe."
        fi

        read -p "¿Quieres eliminar otra máquina virtual? (s/n): " eleccion
        if [[ $eleccion != "s" ]]; then
            break
        fi
    done
}

menu() {
    while true; do
        echo "Gestión de Máquinas Virtuales KVM"
        echo "1) Crear máquina virtual"
        echo "2) Eliminar máquina virtual"
        echo "3) Salir"
        read -p "Elige una opción: " opcion

        case $opcion in
            1)
                crear_mv
                ;;
            2)
                borrar_mv
                ;;
            3)
                echo "Saliendo del script..."
                exit 0
                ;;
            *)
                echo "Opción no válida. Inténtalo de nuevo."
                ;;
        esac
    done
}

#Ejecutar el menú
menu 
~~~  


