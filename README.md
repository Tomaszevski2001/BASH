# BASH
SCRIPT

#!/bin/bash
#0 START
#6N oznacza ze chcemy dokaldnosc mikrosekundowa
startTime=$(date +'%S.%6N')

if [ $# == 0 ]
then
	echo "brak argumentow, sprobuj: $0 <KAT_BAZOWY> <nazwa_pliku_danych1> <nazwa_pliku_danych2> <nazwa_pliku_danychN>" 1>&2
	exit 1
elif [ $# == 1 ]
then
	echo "podano tylko jeden argument, sprobuj: $0 <KAT_BAZOWY> <nazwa_pliku_danych1> <nazwa_pliku_danych2> <nazwa_pliku_danychN>" 1>&2
	exit 2
fi

#petla czaynajaca iterowac od drugiego argumentu
for test_file in "${@:2}"
do
	#sprawdzenie czy pliku podane jako argumenty (zaczynajac od 2 argumentu) istnieja
	if [[ ! -f $test_file ]]
	then
		echo "error, blad dotyczy pliku: $test_file" 1>&2
		exit 3
	fi
done
#1 START
#sprawdzenie czy istnieje katalog o nazwie podanje jako pierwszy argument
# jezeli nie, program utworyz taki katalog

if [ ! -d $1 ]
then
	mkdir "./$1"
fi

#ponowne sprawdzenie czy katalogBazowy istnieje, jesli nie: program konczy prace z kodem bled nr.4
if [ ! -d $1 ]
then
	echo "error, blad dotyczy pliku bazowego" 1>&2
	exit 4
fi

#iterujemy po wszystkich argumetach
for plik in "${@:2}"
do
	#sprawdzamy czy aktyalny argument jest plikiem
	if [[ -f $plik ]]
	then
		while read line
		do
			#polecenie tr z przelacznikie -d pozwala pozbyc sie znaku cudzyslowia
			rok=$(echo $line | cut -d "," -f 3 | tr -d '"')
			miesiac=$(echo $line | cut -d "," -f 4 | tr -d '"')

			if [ ! -d "$1/$rok/$miesiac" ]
			then
				mkdir -m 750 -p "$1/$rok/$miesiac"
			fi
			#2 START
			dzien=$(echo $line | cut -d "," -f 5 | tr -d '"')
			smdb=$(echo $line | cut -d "," -f 7 | tr -d '"')

			if [[ $smdb -ne 8 ]]
			then
				#sprawdzenie czy istnieje plik csv odpowiadajacy danemu dniu
				if [ ! -f "$1/$rok/$miesiac/$dzien.csv" ]
				then     
					#jezeli nie istnieje to tworzymy plik dla danego dnia
					touch "$1/$rok/$miesiac/$dzien.csv"
				fi 
				echo "$line">>"$1/$rok/$miesiac/$dzien.csv"
			fi 
#2 STOP
#3 START

			if [[ $smdb -eq 8 ]]
			then
				if [[ ! -f "$1/$rok.$miesiac.errors" ]]
				then
					touch "$1/$rok.$miesiac.errors"
				fi
				echo "$line">>"$1/$rok.$miesiac.errors"
			fi
		done < "$plik"
#3 STOP
	fi
done



OpadyMax="0"
OpadyMax_file=""
# DO POPrawy 
OpadyMin="100000000"
OpadyMin_file=""
# DO POPRAWY
for entry in "./$1"/*/*/*.csv
do
	suma="0"
	if [[ -f $entry ]]
	then
		while read line
		do
			opady=$(echo $line | cut -d "," -f 6)
			opady=$(echo $opady | sed -e 's/\.//g')
			suma=$((suma + opady))
		done < "$entry"
		
		if [[ $suma -gt $OpadyMax ]]
		then
			OpadyMax=$suma
			OpadyMax_file=$entry
		fi
		if [[ $suma -lt $OpadyMin ]]
		then
			OpadyMin=$suma
			OpadyMin_file=$entry
		fi
	fi
done


#4 START	

	#-m  uprawniena do pliku
	#mkdir -m 750 -p "$1/$rok/$miesiac"


#echo "Opady maksymalne w pliku $OpadyMax_file w pliku wynosza $OpadyMax"
#echo "Opady minimalne w pliku $OpadyMin_file wynosza $OpadyMin"

mkdir "$1"/LINKS


OpadyMax_LINK=$( realpath --relative-to=$1/LINKS $OpadyMax_file )
OpadyMin_LINK=$( realpath --relative-to=$1/LINKS $OpadyMin_file )



#relmax=$(realpath --relitive-to $O)

ln -s "$OpadyMax_LINK" ./$1/LINKS/MAX_OPAD
ln -s "$OpadyMin_LINK" ./$1/LINKS/MIN_OPAD


#4 STOP

stopTime=$(date +'%S.%6N')

#czas=$(echo "scale=0; ($stopTime-$startTime)*1000000/1" | bc)

# Musimy zrobić tak że należy przekoczyć bo napzykład jezeli mamy 3 sekunde 
#minuty to wtedy przeskakuje 


#czas=$(echo "($czas+60000000)/1000" | bc)
#czas=$(echo "czas/1000" | bc)

#miktosekunda ma dokalosc 6 miejsc po przecinku
#-n po echo sprawia ze nie ma znaku konca lini, i nastepne echo bedzie w tej samej linijce
#najpierw wyciagamy sam pid procesu o nazwie poprawka2.sh, pozniej wyciagamy wszystkie pidy i ppidy i grepujemy to z pidem ktorego szukamy

#do pliku trzeba zapisac: PID,PPID,CZAS,WIERSZ_POLECENIA
#pid=$(ps -a -o pid | grep $(ps -a | grep poprawka2.sh | cut -d " " -f 1 | head -n 1) | head -n 1)

#ta zmienna przechowuje pid i ppid
#pidIppid=$(ps -a -o pid,ppid | grep $(ps -a | grep poprawka2.sh | cut -d " " -f 1 | head -n 1) | head -n 1)

# z powyzszej zmienne wyciagam druga kolumne ktora jest ppid
#ppid=$(echo $pidIppid | cut -d " " -f 2)

 

if [ ! -f "$1/out.log" ]
then
        touch "$1/out.log"
fi


echo -n "$BASHPID, "  >> "$1/out.log"

echo -n "$PPID," >> "$1/out.log"

#trzeba jeszcze dodac czas ktory u mnie na kompie niestety nie dziala
echo -n "$czas," >> "$1/out.log"
# $@ to komenda wpisana do wiersza polecen podczas odpalana skryptu
echo $@ >> "$1/out.log"

#0 STOP
exit 0
