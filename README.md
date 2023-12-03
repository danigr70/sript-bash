# sript-bash
#!/bin/bash

# Verificar si el usuario es root
if [ "$(whoami)" != "root" ]; then
    echo "Este script debe ser ejecutado por el usuario root."
    exit 1
fi

# Ruta del archivo 
archivo="paquetes.txt"

# Verificar si el archivo existe
if [ ! -e "$archivo" ]; then
    echo "El archivo $archivo no existe."
    exit 1
fi

# Función para verificar si un paquete está instalado
paquete_instalado() {
    local paquete="$1"
    local buscar_paquete_actual=$(whereis "$paquete" | grep bin | wc -l)

    if [ "$buscar_paquete_actual" -eq 0 ]; then
        echo "DEBUG: El paquete $paquete no está instalado."
        return 1
    else
        echo "DEBUG: El paquete $paquete está instalado."
        return 0
    fi
}
# Leer el archivo y cargar los datos en arrays
mapfile -t paquetes < "$archivo"

# Procesar cada línea de los arrays
for indice in "${!paquetes[@]}"; do
    IFS=":" read -r paquete accion <<< "${paquetes[$indice]}"

    case $accion in
        add | a)
            if paquete_instalado "$paquete"; then
                echo "El paquete $paquete ya está instalado."
            else
                echo "El paquete $paquete se instalará."
                sudo apt install -y "$paquete"
                sudo snap install brave
            fi
            ;;
        remove | r)
            if paquete_instalado "$paquete"; then
                echo "Desinstalando el paquete $paquete"
                sudo apt remove -y "$paquete"
                sudo apt purge -y "$paquete"
            else
                echo "El paquete $paquete no está instalado."
            fi
            ;;
        status | s)
            paquete_instalado "$paquete"
                echo "status del $paquete"
                sudo systemctl status "$paquete"
            ;;
        *)
            echo "Operación desconocida para el paquete $paquete: $accion"
            ;;
    esac
done
