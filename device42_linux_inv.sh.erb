#!/bin/bash -e

############################################################
#
# Author: John Alberts
# Creation Date: Oct 29, 2012
# Last Modified: Nov 5, 2012
#
# Description: Detects hardware and some custom info and sends
# it to a device42 server.
#
# Notes: Only tested on RHEL4 and CentOS5 & 6 servers.
# Requires at least device42 v322
#
# Changelog:
# v1.0 - 11/05/2012 (John Alberts) - Initial release
# v1.0.1 - 11/07/2012 (John Alberts) - Some error handling
#
############################################################


VERSION="1.0.1"
DEBUG=true

### Edit lines below
### Begin Custom
USER="user"
PASS="password"
D42SERVER="https://your_device42_server_here"
##############################################
# Used for sending email when errors occur
MAIL="$(which sendmail)"
TO="you@email.com"
SUBJECT="[DEVICE42 ERROR] -"
EMAIL="TO:${TO}
FROM:device42_inventory_script@noreply.com
"
##############################################
DNAMEFILTER="s/\.domain\.\(com\|int\)//g"

### End Custom



DEVICEURL="${D42SERVER}/api/device/"
IPURL="${D42SERVER}/api/ip/"
CFURL="${D42SERVER}/api/1.0/device/custom_field/"
MACURL="${D42SERVER}/api/1.0/macs/"



function email_error() {
  # $1 = Error exit status
  # $2 = Message

  SUBJECT="${SUBJECT} Exit Status: ${1}"
  EMAIL="SUBJECT: ${SUBJECT}\n${EMAIL}\n${2}"
  echo -e "${EMAIL}" | ${MAIL} -t
}

# trap handler: print location of last error and process it further
#
function my_trap_handler() {
  MYSELF="$0"               # equals to my script name
  LASTLINE="$1"            # argument 1: last line of error occurence
  LASTERR="$2"             # argument 2: error code of last command
  echo "script error encountered at `date` in ${MYSELF}: line ${LASTLINE}: exit status of last command: ${LASTERR}"

  # do additional processing: send email or SNMP trap, write result to database, etc.
  email_error "${LASTERR}" "${LASTLINE}"
}

trap 'my_trap_handler ${LINENO} $?' 1 3 5 15

DATA=""

function urlencode() {
  echo -ne "${1}" | xxd -plain | tr -d '\n' | sed 's/\(..\)/%\1/g'
}

function add_data() {
  DATA="${DATA}&${1}"
}

function post_data() {
# expects a data string to be passed as the 1st parameter.
  # $1 = data to send
  # $2 = URL to send to
  if echo "${2}" | grep '/custom_field/' 2>&1 >/dev/null; then
    REQ="PUT"
  else
    REQ="POST"
  fi
  thisdata="$(echo -ne "${1}" | sed 's/:/%3A/g')"
  [[ $DEBUG ]] && echo "Sending: ${2}${thisdata}"
  STATUS="$(curl -k -i -X ${REQ}  --data "${thisdata}" -u "${USER}:${PASS}" "${2}" 2>&1)"
  if ! echo "${STATUS}" | grep '"code": 0' 2>&1 >/dev/null; then
    echo "Failed to send the following data to the server"
    echo "${thisdata}"
    echo -e "Curl Status: ${STATUS}"
    echo "Exiting"
    email_error "post data error" "${STATUS}"
    exit 1
  fi
  [[ $DEBUG ]] && echo -e "Sent Data: ${2}${thisdata}\n"
}


# Get device name
DNAME="$(hostname -f | sed "${DNAMEFILTER}")"
# This is to fix hosts that didn't have their hostname set properly
if ! echo "${DNAME}" | grep '\.' 2>&1 >/dev/null; then
  DNAME="${DNAME}.hosted"
fi
DATA="name=${DNAME}"

# Get Total RAM
MEMORY_TOTAL="$(( $(sed -n -s 's/^MemTotal:\s\+\([0-9]\+\) .*$/\1/p' /proc/meminfo) / 1024 ))"
if [[ $MEMORY_TOTAL -lt 512 ]]; then
  x=128
elif [[ $MEMORY_TOTAL -lt 1024 ]]; then
  x=256
elif [[ $MEMORY_TOTAL -lt 4096 ]]; then
  x=512
elif [[ $MEMORY_TOTAL -lt 8192 ]]; then
  x=1024
else
  x=2048
fi
MT=$(( $x * ( $MEMORY_TOTAL / $x ) ))
add_data "memory=${MT}"

# Get OS
if [[ -f /etc/redhat-release ]]; then
  OS="$(sed 's/^\(.\+\) release.*/\1/' /etc/redhat-release | tr '[A-Z]' '[a-z]')"
  OSVERSION="$(sed 's/^.\+ release \([.0-9]\+\).*/\1/' /etc/redhat-release)"
elif [[ -f /etc/lsb-release ]]; then
  OS="$(grep DISTRIB_ID /etc/lsb-release | cut -d "=" -f 2 | tr '[A-Z]' '[a-z]')"
  OSVERSION="$(grep DISTRIB_RELEASE /etc/lsb-release | cut -d "=" -f 2)"
fi
add_data "os=${OS:-other}&osver=${OSVERSION:-other}"

# Get system info
# Check if virtual server
VIRTUAL=0
IS_VSERVER_GUEST=no
IS_VIRTUAL_HOST=no
IS_VIRTUAL_GUEST=no
VIRTUAL_HOST=
DMIDECODE="$(which dmidecode 2>/dev/null)"
if [[ -e /proc/virtual ]]; then
  VIRTUAL="VServer host"
  IS_VIRTUAL_HOST="yes"
elif [[ ! -e /proc/virtual && ! -e /dev/mem ]]; then
  VIRTUAL="VServer guest"
  IS_VIRTUAL_GUEST="yes"
  IS_VSERVER_GUEST="yes"
  CPUCOUNT="$(grep 'cpu[0-9]' /proc/stat | wc -l)"
  CPUSPEED="$(sed -n 's/^model name\s\+:\s\+.*@\s*\([0-9.]\+\).*$/\1/p' /proc/cpuinfo | head -n1)"
  CPUSPEEDUNIT="$(sed -n 's/^model name\s\+:\s\+.*@\s*[0-9.]\+\s*\([a-zA-Z]\+\)$/\1/p' /proc/cpuinfo | head -n1 | tr '[A-Z]' '[a-z]')"
  CPUCORES="1"
  case "${CPUSPEEDUNIT}" in
    ghz)
      multiplier="1000"
      ;;
    mhz)
      multiplier="1"
      ;;
    hz)
      multiplier="0.001"
      ;;
  esac
  CPUSPEEDMHZ="$(echo "${CPUSPEED} * ${multiplier}" | bc)"
  VIRTUAL_HOST="$(cat /etc/vservers/host.txt 2>/dev/null).hosted"
fi

if [[ "${IS_VSERVER_GUEST}" == "no" ]]; then
  if [[ "${DMIDECODE}" != "" ]]; then
    SERIAL="$(${DMIDECODE} -s system-serial-number)"
    PRODUCTNAME="$(${DMIDECODE} -s system-product-name)"
    MANUFACTURER="$(${DMIDECODE} -s system-manufacturer)"
    case "${PRODUCTNAME}" in
      "VMware, Inc."|Bochs|KVM|QEMU|"Microsoft Corporation"|Xen)
      MANUFACTURER="vmware"
      VIRTUAL="${PRODUCTNAME}"
      IS_VIRTUAL_GUEST="yes"
      ;;
    esac
    CPUSPEEDMHZ="$(${DMIDECODE} -t processor | sed -n 's/^\s\+Max Speed:\s\+\([0-9]\+\).*$/\1/p' | head -n1)"
    CPUCORES="$(${DMIDECODE} -t processor | sed -n 's/^\s\+Core Count:\s\+\([0-9]\+\).*$/\1/p' | head -n1)"
    CPUCOUNT="$(${DMIDECODE} -t processor | grep 'Socket Designation' | wc -l)"
  else
    echo "dmidecode is not installed."
    echo "some hardware information will not be collected"
  fi
fi
if [[ "$VIRTUAL" != "0" && "$VIRTUAL" != "VServer host" ]]; then
  MANUFACTURER="vmware"
else
  if lsmod | grep 'kvm' 2>&1 >/dev/null; then
    IS_VIRTUAL_HOST="yes"
  fi
fi

if [[ "${IS_VIRTUAL_GUEST}" == "yes" || "${IS_VSERVER_GUEST}" == "yes" ]]; then
  TYPE="virtual"
  SERIAL="virtual"
  [[ ${PRODUCTNAME} ]] || PRODUCTNAME="virtual"
  [[ ${MANUFACTURER} ]] || MANUFACTURER="virtual"
else
  TYPE="physical"
fi

add_data "manufacturer=${MANUFACTURER%% *}&serial_no=${SERIAL%% *}&hardware=${PRODUCTNAME%% *}&cpucount=${CPUCOUNT:-1}&cpupower=${CPUSPEEDMHZ%%.*}&cpucore=${CPUCORES:-1}&type=${TYPE}"
if [[ "${IS_VIRTUAL_HOST}" == "yes" ]]; then
  add_data "is_it_virtual_host=yes"
fi

[[ ${VIRTUAL_HOST} ]] && add_data "virtual_host=${VIRTUAL_HOST}"

# Send the data we have so far to the device42 server
echo "Sending hardware data to server"
post_data "${DATA}" "${DEVICEURL}"


# Now, send the nic info
# First, we need to get the device number for this server

DATA=""
NICS="$(ifconfig | egrep -A1 '^[a-zA-Z]')"
OLDIFS="${IFS}"
IFS=$'\n'
currentnic=""
thismac=""
thisip=""
echo "Sending NIC info to server"
for thisline in ${NICS}; do
  if echo "${thisline}" | egrep '^[a-zA-Z]' 2>&1 >/dev/null; then
    # This must be the first line wtih the nic name
    currentnic="$(echo "${thisline}" | sed -n 's/^\([[:alnum:]]\+\).*$/\1/p')"
    thismac="$(echo "${thisline}" | sed -n 's/^.*\s\+HWaddr\s\+\([0-9A-Z:]\+\)\s*$/\1/p')"
  elif echo "${thisline}" | egrep '^[[:space:]]+' 2>&1 >/dev/null; then
    thisip="$(echo "${thisline}" | sed -n 's/^\s\+inet addr:\([0-9.]\+\).*$/\1/p')"
  elif echo "${thisline}" | grep '\-\-' 2>&1 >/dev/null; then
    # This must be the end of the info for this nic, so send the data
    if [[ "$thisip" != "" ]]; then
      DATA="device=${DNAME}&tag=${currentnic}&ipaddress=${thisip:-none}"
      if [[ "$thismac" != "" && "$thismac" != "00:00:00:00:00:00" ]]; then
        DATA="${DATA}&macaddress=${thismac}"
      fi
      post_data "${DATA}" "${IPURL}"
    fi
    if [[ "$thismac" != "" && "$thismac" != "00:00:00:00:00:00" ]]; then
      DATA="device=${DNAME}&macaddress=${thismac}&port_name=${currentnic}"
      post_data "${DATA}" "${MACURL}"
    fi
  fi
done

# Set custom fields
DATA=""
DATA="name=${DNAME}&key=test&value=test_value"
CF="$(egrep 'CustomDNS|CustomerCode|CustomerName|CustomerRegion|DecomissionDate|InstallationTicketNumber|ProductTotalCareOrDirect|ProductionState|ServerPrimaryContact|ServiceContractVendor|ImplentationServer|ApplicationBackupsConfigured|DracIPAddress|BackupNetowrkIPAddress|NetbackupClientInstalled|NetappID' /etc/.exlhs/.server_info.txt)"
for thiscf in ${CF}; do
  thiskey=${thiscf%%:*}
  thisvalue="$(urlencode "${thiscf#*:}")"
  post_data "name=${DNAME}&key=${thiskey}&value=${thisvalue:-none}" "${CFURL}"
done

IFS="${OLDIFS}"

exit 0
