- **写在前面：该spark是客户端，是和openfire服务器一起使用的，不是那个spark集群~**。  
- 对于spark客户端的启动流程和openfire启动流程所涉及的知识没有多大区别，（部分涉及知识请参照 [openfire服务器源码阅读学习之启动流程（一）](https://blog.csdn.net/qq_38172320/article/details/90036025)）只是新加了unpackArchives(File libDir, boolean printStatus)函数；  

- unpackArchives(File libDir, boolean printStatus)函数解释：将目录中的任何包文件转换为标准JAR文件。各包文件在转换为JAR后将被删除。如果没有找到包文件，此方法不起任何作用。  
	代码如下：
	
		private void unpackArchives(File libDir, boolean printStatus) {
        	//获取一个libDir文件夹下的所有文件，并且根据.pack后缀过滤文件
        	File[] packedFiles = libDir.listFiles( ( dir, name ) -> {
            	return name.endsWith(".pack");
       		 } );
		     // 如果找不到文件，则返回，不解压
       		 if (packedFiles == null) {
            	// Do nothing since no .pack files were found
            	return;
        	}
			//找到文件，解压
        	boolean unpacked = false;
        	for (File packedFile : packedFiles) {
	            try {
        	        String jarName = packedFile.getName().substring(0,
        	            packedFile.getName().length() - ".pack".length());
        	        File jarFile = new File(libDir, jarName);
        	          // 删除重复的jar包
        	        if (jarFile.exists()) {
        	            jarFile.delete();
        	        }
	  	    //将.pack文件转换为jar文件
            InputStream in = new BufferedInputStream(new FileInputStream(packedFile));
            JarOutputStream out = new JarOutputStream(new BufferedOutputStream(
            	new FileOutputStream(new File(libDir, jarName))));
                Pack200.Unpacker unpacker = Pack200.newUnpacker();
                // 打印解压状态
                if (printStatus) {
                    System.out.print(".");
                }
                // 解压，写入到out里
                unpacker.unpack(in, out);
                in.close();
                out.close();
                //删除压缩文件
                packedFile.delete();
                unpacked = true;
            }
            catch (Exception e) {
                e.printStackTrace();
            }
        }
        	// Print newline if unpacking happened.
        	if (unpacked && printStatus) {
       		     System.out.println();
       		 }
       	}
   
    
    
 
注：pack200：这个是JDK中自带的一个压缩工具，能够对普通的jar文件进行高效压缩。这个压缩工具压缩其他文件和普通压缩工具没有区别，但是压缩jar文件将能轻易达到10%~40%的压缩率。


