#!/bin/bash

# Script for backing up local MySQL database
# It's good to use it with Cron tasks
# Author: ceskyDJ (https://github.com/ceskyDJ)

# Control vars defaults definition
force=false;
remove=false;
time=false;
user="root";
password="";
validity="31";

# Load config from arguments 
i=0; # Iteration
skip=false; # Sign for skip next argument (because of value)

for arg in $@;
do
	if [ $skip == true ]; then
		skip=false;
		i=$(echo "$i + 1"|bc);
		continue;
	fi

	case $arg in
	# Display help
	"-h" | "--help")
		echo "Usage: $0 DIRECTORY [OPTIONS...]";
		
		echo -e "\nCommand structure:";
		echo -e "  DIRECTORY\t\t\tdirectory for saving backups";
		echo -e "  OPTIONS\t\t\tadditional arguments";
	
		echo -e "\nAvailable arguments:";
		echo -e "  -f, --force\t\t\tforce rewrite backup file";
		echo -e "  -r, --remove\t\t\tremove old backup files (after month)";
		echo -e "  -t, --time\t\t\tadd time to backup file names";
		echo -e "  -u USER, --user=USER\t\tset database user (with full read permissions) - default: 'root'";
		echo -e "  -p PASS, --password=PASS\tset password for database user - default: ''";
		echo -e "  -v DAYS, --validity=DAYS\tset backup file validity in days (useful only with remove argument) - default: '31'";
		echo -e "  -h, --help\t\t\tdisplay this help";
	
		exit;
		;;
	# Allow backup rewriting
	"-f" | "--force")
		force=true;
		;;
	# Allow old backup files removing
	"-r" | "--remove")
		remove=true;
		;;
	# Adding time
	"-t" | "--time")
		time=true;
		;;
	# User change
	"-u")
		user=${@:$i+2:1}; # +1 (it starts from 1), +1 (move to next iteration index)
		skip=true;
		;;
	--user=*)
		user=$(echo $arg | cut -b 8-);
		;;
	# Password change
	"-p")
		password=${@:$i+2:1}; # +1 (it starts from 1), +1 (move to next iteration index)
		skip=true;
		;;
	--password=*)
		password=$(echo $arg | cut -b 12-);
		;;
	# Backup validity change
	"-v")
		validity=${@:$i+2:1}; # +1 (it starts from 1), +1 (move to next iteration index)
		skip=true;
		;;
	--validity=*)
		validity=$(echo $arg | cut -b 12-);
		;;
	# Bad argument
	*)
		# First argument + valid directory
		if [ $i == 0 ] && [ -d $arg ]; then
			dir=$arg;
		# First argument + not existing directory
		elif [ $i == 0 ] && [ ! -d $arg ]; then
			echo "Directory you specified is wrong. Enter valid directory or use -h or --help argument for display help.";
			
			exit;
		# Second and next argument
		else
			# Invalid argument
			echo "You entered bad argument '$arg'. Try -h or --help argument to display help.";
		
			exit;
		fi
		;;
	esac
	
	i=$(echo "$i + 1"|bc);
done

# Directory verification
if [ ! $dir ]; then
	echo "Directory you specified is wrong. Enter valid directory or use -h or --help argument for display help.";
	
	exit;
fi

# Date and time format setting
if [ $time == true ]; then
	format="%Y-%m-%d_%T";
else
	format="%Y-%m-%d";
fi

# Backup creation
backupFile="$dir/database_`date +$format`.sql.gz";

# Only if backup wasn't created (or rewrite is allowed)
if [ $force == true ] || [ ! -f $backupFile ]; then
	mysqldump -u $user --password=$password --all-databases | gzip > $backupFile
fi

# Old backups removing
if [ $remove == true ]; then
	for file in $dir/*.sql.gz; do
		length=$(expr length $dir); # -> length of directory string
	
		start=$(echo "11 + $length"|bc); # -> start index
		end=$(echo "20 + $length"|bc); # -> end index
		dateString=$(echo $file | cut -b $start-$end); # -> YYYY-MM-DD	
	
		dateFormat=$(date -d $dateString +"%Y%m%d"); # -> YYYYMMDD
	
		minimum=$(date -d -$validity+"day" +"%Y%m%d");

		if [ $minimum -ge $dateFormat ]; then
			rm $file;
		fi
	done
fi
