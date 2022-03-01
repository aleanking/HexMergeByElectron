<template>
  <div id="wrapper">
   <el-upload
     class="bootloader-demo"
     ref="bootloader"
	 action=""
     :limit = "1"
     :on-preview="boothandlePreview"
     :on-remove="boothandleRemove"
     :on-change ="boothandlechange"
     :file-list="bootfileList"
     :auto-upload="false">
     <el-button slot="trigger" type="primary">Select the BootLoader Hex file</el-button>
     <div slot="tip" class="el-upload__tip">CAN ONLY SELECT THE BootLoader TYPE Hex File</div>
   </el-upload>
	<el-upload
	class="upload-demo"
	ref="upload"
	action=""
	:limit = "1"
	:on-preview="handlePreview"
	:on-remove="handleRemove"
	:on-change ="handlechange"
	:file-list="fileList"
	:auto-upload="false">
	<el-button slot="trigger" type="primary">Select Application Hex File</el-button>
	<el-button style="margin-left: 10px;" type="success" @click="changehex">MERGE</el-button>
	<div slot="tip" class="el-upload__tip">CAN ONLY SELECT THE APPLICATION Hex FILE</div>
	</el-upload>
    <el-progress :text-inside="true" :stroke-width="24" :percentage="changepercentage" status="success"></el-progress>
	<span>THE MERGE FILE IS：{{otafilename}} </span>
  </div>
</template>
<script>
var fs = require('fs');
var readline = require('readline');
var destfilepath = '';
var tempfilepath = '';
   export default {
      data() {
        return {
          fileList: [],
		  srcfile:'',
		  bootfileList: [],
		  bootsrcfile:'',
		  changepercentage:0,
		  otafilename:''
        };
      },
      methods: {
        changehex() {
		  if((this.srcfile == '')||(this.bootsrcfile == ''))
		  {
			  this.$message.error('SELECT the bootlaoder and application hex file');
			  return;
		  }
		  this.changepercentage =0;
		  console.log(this.srcfile );
		  var that =this;
		  var fWriteName = tempfilepath;
		  var fWrite = fs.createWriteStream(fWriteName);
		  
		  const rl = readline.createInterface({
		    input: fs.createReadStream(that.bootsrcfile),
		    crlfDelay: Infinity
		  });
		  var totalline =0;
		  rl.on('line', (line) => {		   
		    totalline++;		
		  });	  
		  rl.on('close', ()=>{
		  	const rrl = readline.createInterface({
		  				input: fs.createReadStream(that.bootsrcfile),
		  				crlfDelay: Infinity
		  				});
		  	var rtotalline =0;
		  	rrl.on('line', (line) => {
				that.changepercentage = rtotalline/(totalline-1)-5				
		  		rtotalline++;
		  		if(rtotalline < totalline-1)
		  		  fWrite.write(`${line}`+'\r\n');
		  	});
		  	rrl.on('close', ()=>{		  		
		  		function myFunc(arg) {
		  		  	var contentText = fs.readFileSync(fWriteName);
		  			var appandcontentText = fs.readFileSync(that.srcfile);
		  			var destfilebuffer =  Buffer.concat([contentText,appandcontentText]);
		  			fs.writeFile(destfilepath, destfilebuffer, function(err){
						that.changepercentage =100;	
		  				if(err){
							this.$message.error('merge failed');
		  				}else {
							that.$message({
							          message: 'merge succes',
							          type: 'success'
							        });
							that.otafilename = destfilepath
		  				}
						tempfilepath = ''
						destfilepath = ''
		  			})
		  		}		  
		  		setTimeout(myFunc, 1500, 'funky');		  
		  	});  
		  }); 
        },
        handleRemove(file, fileList) {
		  this.srcfile = '';
		  destfilepath = '';
		  this.otafilename = '';
        },
        handlePreview(file) {
        },
		handlechange(file, fileList) {
		  this.srcfile = file.raw.path
		  var ss = this.srcfile.split('\\');
		  for(var i=0; i<ss.length-1;i++)
			destfilepath+= ss[i]+'\\';
		  tempfilepath = destfilepath;
		  tempfilepath+='temp.hex'
		  destfilepath+=ss[ss.length-1].split('.')[0]+'-ota.hex'		  
        },
		
		boothandleRemove(file, fileList) {
		  this.bootsrcfile = '';
		  this.otafilename = '';
		  destfilepath = '';
		},
		boothandlePreview(file) {
		},
		boothandlechange(file, fileList) {
		  this.bootsrcfile = file.raw.path  
		},
      }
    }
</script>

<style>
 
</style>
