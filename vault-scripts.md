vault.get  - get aws credential
```
#!/bin/sh
#  vault.get
#       get your aws credential using vault
if [ $# -ne 2 ] ; then
   echo "Usage: $0 ENV TEAM"
   echo "Where ENV - dev, nonprd, prod"
   echo "      TEAM - cia, ps, fss, mais, dmr, ps"
fi

export VAULT_ADDR=https://your-vault-server
echo "export VAULT_ADDR=$VAULT_ADDR"
ENVI=$1
TEAM=$2

case $ENVI in
dev|nonprod|prod)
    echo "Executing vault read aws-asia-${ENVI}/creds/$TEAM ..."
    vault read aws-asia-${ENVI}/creds/$TEAM | grep _key > /tmp/logfile
    AWS_ACCESS_KEY_ID=`cat /tmp/logfile | grep access_key | awk '{print $2}'`
    AWS_SECRET_ACCESS_KEY=`cat /tmp/logfile | grep secret_key | awk '{print $2}'`
    ;;
*)
   echo "$0 ENV TEAM"
   echo "Where ENV - dev, nonprd, prod"
   echo "      TEAM - cia, ps, fss, mais, dmr, ps"
   ;;
esac

export AWS_DEFAULT_REGION=us-west-2
export AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_DEFAULT_REGION
echo "exporting AWS_DEFAULT_REGION, AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY"

```
vault.login - login to vault
```
#!/bin/bash
export VAULT_ADDR=https://your-vault-server
echo "Exporting VAULT_ADDR=$VAULT_ADDR"
echo "Executing valut login -method=ldap username whoami"
vault login -method=ldap username=`whoami`
```
connect.sh - ssh to aws ec2 using tag Name
```
#!/bin/bash
if [ $# -ne 2 ] ; then
   echo "$0 HN Key"
   exit 1
fi

EC2=ec2.list
HN=$1
KP=$2
echo "Looking for public IP with tag name $HN ..."
aws ec2 describe-instances  --query 'Reservations[].Instances[].[PrivateIpAddress,PublicIpAddress,VpcId,InstanceId,Tags[?Key==`Name`].Value[]]' --output text --filters "Name=instance-state-name,Values=running" | sed 's/None$/None\n/g' | sed '$!N;s/\n/ /g' | nl > $EC2

IP=`cat $EC2 | grep $HN | awk '{print $3}'`
echo "Found IP $IP of $HN"

rm -f $EC2

echo "Connecting to $HN with $IP ..."
ssh -i $KP admin\@$IP
