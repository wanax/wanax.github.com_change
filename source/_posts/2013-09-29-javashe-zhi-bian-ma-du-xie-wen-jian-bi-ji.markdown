---
layout: post
title: "java设置编码读写文件笔记"
date: 2012-11-01 16:04
comments: true
categories: Tec
---
``` java
public class Test {  
        public static void main(String[] args) {  
            try{  
                BufferedReader br = new BufferedReader(  
                        new InputStreamReader(new FileInputStream(url),"UTF-8"));  
                String data = null;  
                StringBuffer str=new StringBuffer();  
                while((data = br.readLine())!=null)  
                {  
                    str.append(data+"\n");  
                }  
                System.out.println(str);  
                br.close();  
                Test test=new Test();  
                test.ece(str.toString());  
            }catch(Exception e){  
                e.printStackTrace();  
            }             
        }  
        public void ece(String str){  
            try{  
                BufferedWriter bw = new BufferedWriter(  
                        new OutputStreamWriter(new FileOutputStream(url,false),"UTF-8"));  
                bw.write(str);  
                bw.flush();  
                bw.close();  
            }catch(Exception e){  
                e.printStackTrace();  
            }     
        }  
} 
```