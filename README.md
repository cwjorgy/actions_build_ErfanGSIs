name: build_ErfanGSIs

on:
#  release:
#    types: [published]
#  push:
#    branches:
#      - master
#    paths:
#      - '.config'
#  schedule:
#    - cron: 0 8 * * 5
  watch:
    types: [started]
    
env:
  ROM_URL: https://hugeota.d.miui.com/20.5.29/miui_CEPHEUS_20.5.29_d531f234ac_10.0.zip
  ROM_NAME: MIUI
  TZ: Asia/Shanghai

jobs:
  build:
    runs-on: ubuntu-latest
    if: github.event.repository.owner.id == github.event.sender.id

    steps:
       - name: Checkout
         uses: actions/checkout@master
       
       - name: Clean Up
         run: |
           docker rmi `docker images -q`
           sudo rm -rf /usr/share/dotnet /etc/mysql /etc/php /etc/apt/sources.list.d
           sudo -E apt-get -y purge azure-cli ghc* zulu* hhvm llvm* firefox google* dotnet* powershell openjdk* mysql* php*
           sudo -E apt-get update
           sudo -E apt-get -y autoremove --purge
           sudo -E apt-get clean 
           
       - name: Initialization environment
         run: |
            sudo -E apt-get -qq update
            sudo -E apt-get -qq install git openjdk-8-jdk wget
       
       - name: Clone ErfanGSI Source Code

         run: git clone --recurse-submodules https://github.com/qsf1415/ErfanGSIs.git

       

       - name: Setting up ErfanGSI requirements

         run: |

              sudo chmod -R 777 ErfanGSIs

              cd ErfanGSIs

              sudo bash setup.sh

              sudo sed -i '7c AB=true' url2GSI.sh

              sudo sed -i '8c Aonly=false' url2GSI.sh

       

       - name: Download Stock Rom & Generate GSI

         run: sudo ./ErfanGSIs/url2GSI.sh $ROM_URL $ROM_NAME
       
       - name: Zip GSI  
         run: |
              mkdir final
              zip -r final/GSI.zip /home/runner/work/actions_build_ErfanGSIs/actions_build_ErfanGSIs/ErfanGSIs/output/
       
       - name: Upload GSI to BitSend
         run: |
              curl -sL https://git.io/file-transfer | sh
              ./transfer bit final/GSI.zip

         
            
