De nuevo Jose te dejo los bloques de scripts por aquí para que te sean más visibles.

## EJERCICIO 1
~~~
#!/bin/bash
# EJERCICIO 1 DE LA TAREA 10 SCRIPTS KVM - CREACIÓN DE POOLS - ALEJANDRO LAMPREA PÉREZ


# Función para visualizar los pools existentes

visualizar_pools() {
    echo "Pools de almacenamiento existentes:"
    virsh pool-list --all
}

# Función para configurar un pool

configurar_pool() {
    while true; do
        echo "Configurar un pool de almacenamiento"
        echo "Opciones:"
        echo "a) Crear discos duros"
        echo "b) Mostrar información de los discos duros"
        echo "c) Agregar discos duros a una máquina virtual"
        echo "d) Definir, inicializar y habilitar arranque automático de un pool"
        read -p "Elige una opción (a/b/c/d): " opcion

        case $opcion in
            a)
                read -p "Número de discos duros a crear: " numero_discos
                for ((i=1; i<=numero_discos; i++)); do
                    read -p "Nombre del disco $i: " nombre_disco
                    read -p "Tamaño del disco (en GB): " tamano_disco
                    qemu-img create -f qcow2 /var/lib/libvirt/images/"$nombre_disco".qcow2 "${tamano_disco}G"
                    echo "Disco $nombre_disco creado con éxito."
                done
                ;;
            b)
                echo "Información de los discos duros en /var/lib/libvirt/images:"
                ls -lh /var/lib/libvirt/images
                ;;
            c)
                read -p "Nombre de la máquina virtual: " nombre_maquina
                read -p "Ruta del disco duro a agregar (ejemplo: /var/lib/libvirt/images/disk.qcow2): " ruta_disco
                virsh attach-disk "$nombre_maquina" "$ruta_disco" vdb --persistent
                echo "Disco $ruta_disco agregado a $nombre_maquina."
                ;;
            d)
                read -p "Nombre del pool: " nombre_pool
                read -p "Ruta del directorio para el pool: " ruta_pool
                mkdir -p "$ruta_pool"
                virsh pool-define-as "$nombre_pool" dir --target "$ruta_pool"
                virsh pool-build "$nombre_pool"
                virsh pool-start "$nombre_pool"
                virsh pool-autostart "$nombre_pool"
                echo "Pool $nombre_pool configurado, inicializado y con arranque automático."
                ;;
            *)
                echo "Opción no válida."
                ;;
        esac

        read -p "¿Quieres realizar otra operación en configuración de pools? (s/n): " continuar
        if [[ $continuar != "s" ]]; then
            break
        fi
    done
}

# Función para borrar pools

borrar_pools() {
    while true; do
        visualizar_pools
        read -p "Introduce el nombre del pool a borrar: " nombre_pool
        virsh pool-destroy "$nombre_pool" &>/dev/null
        virsh pool-undefine "$nombre_pool"
        echo "Pool $nombre_pool borrado con éxito."

        read -p "¿Quieres borrar otro pool? (s/n): " continuar
        if [[ $continuar != "s" ]]; then
            break
        fi
    done
}

# Función para generar un informe de pools

generar_informe() {
    informe="Informe_Pool.txt"
    echo "Generando informe..."
    echo "Pools de almacenamiento - Informe" > "$informe"
    echo "--------------------------------" >> "$informe"

    total_pools=$(virsh pool-list --all | wc -l)
    pools_activos=$(virsh pool-list | wc -l)
    pools_inactivos=$((total_pools - pools_activos - 2)) # Restar cabeceras y ajuste

    echo "Número total de pools: $((total_pools - 2))" >> "$informe"
    echo "Número de pools activos: $((pools_activos - 2))" >> "$informe"
    echo "Número de pools inactivos: $pools_inactivos" >> "$informe"

    echo "Informe generado: $informe"
    cat "$informe"
}

# Función principal del menú

menu_principal() {
    while true; do
        echo "Gestión de Pools de Almacenamiento en KVM"
        echo "1) Visualizar pools existentes"
        echo "2) Configurar un pool"
        echo "3) Borrar pools"
        echo "4) Generar informe de pools"
        echo "5) Salir"
        read -p "Elige una opción: " opcion

        case $opcion in
            1)
                visualizar_pools
                ;;
            2)
                configurar_pool
                ;;
            3)
                borrar_pools
                ;;
            4)
                generar_informe
                ;;
            5)
                echo "Saliendo del script."
                exit 0
                ;;
            *)
                echo "Opción no válida."
                ;;
        esac
    done
}

# Ejecutar el menú principal
menu_principal
~~~
