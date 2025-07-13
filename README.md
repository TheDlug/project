#!/bin/bash

# Název souboru, do kterého ukládáme úkoly
soubor="todo.txt"

# Pokud soubor neexistuje, vytvoří se
if [ ! -f "$soubor" ]; then
    touch "$soubor"
fi

# Funkce pro zobrazení úkolů
zobraz_ukoly() {
    echo "-------- Seznam úkolů --------"
    cislo=1
    while IFS= read -r radek
    do
        stav=$(echo "$radek" | cut -d ";" -f1)
        text=$(echo "$radek" | cut -d ";" -f2-)

        if [ "$stav" -eq 0 ]; then
            echo "$cislo) [ ] $text"
        else
            echo "$cislo) [x] $text"
        fi

        cislo=$((cislo+1))
    done < "$soubor"
}

# Funkce pro přidání úkolu
pridej_ukol() {
    echo -n "Zadej popis úkolu: "
    read ukol
    if [ -z "$ukol" ]; then
        echo "Nelze přidat prázdný úkol."
    else
        echo "0;$ukol" >> "$soubor"
        echo "Úkol byl přidán."
    fi
}

# Funkce pro označení úkolu jako splněného
oznac_splneny() {
    zobraz_ukoly
    echo -n "Zadej číslo úkolu ke splnění: "
    read cislo

    if ! [[ "$cislo" =~ ^[0-9]+$ ]]; then
        echo "Zadej platné číslo."
        return
    fi

    celkem=$(wc -l < "$soubor")
    if [ "$cislo" -gt "$celkem" ] || [ "$cislo" -lt 1 ]; then
        echo "Úkol s tímto číslem neexistuje."
        return
    fi

    sed -i "${cislo}s/^0/1/" "$soubor"
    echo "Úkol označen jako splněný."
}

# Funkce pro smazání úkolu
smaz_ukol() {
    zobraz_ukoly
    echo -n "Zadej číslo úkolu ke smazání: "
    read cislo

    if ! [[ "$cislo" =~ ^[0-9]+$ ]]; then
        echo "Zadej platné číslo."
        return
    fi

    sed -i "${cislo}d" "$soubor"
    echo "Úkol byl smazán."
}

# Hlavní smyčka s menu
while true
do
    echo ""
    echo "===== TO-DO MENU ====="
    echo "1) Přidat úkol"
    echo "2) Zobrazit úkoly"
    echo "3) Označit úkol jako splněný"
    echo "4) Smazat úkol"
    echo "5) Konec"
    echo -n "Vyber možnost (1-5): "
    read volba

    case $volba in
        1) pridej_ukol ;;
        2) zobraz_ukoly ;;
        3) oznac_splneny ;;
        4) smaz_ukol ;;
        5) echo "Ukončuji..."; break ;;
        *) echo "Neplatná volba, zkus to znovu." ;;
    esac
done
