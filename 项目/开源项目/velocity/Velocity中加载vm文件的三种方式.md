Velocity中加载vm文件的三种方式

velocitypropertiespath
Velocity中加载vm文件的三种方式：
 
方式一：加载classpath目录下的vm文件
Properties p = new Properties();
p.put("file.resource.loader.class",
"org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader");
Velocity.init(p);
...
Velocity.getTemplate(templateFile);
 
 
方式二：根据绝对路径加载，vm文件置于硬盘某分区中，如：d://tree.vm
Properties p = new Properties();
p.setProperty(VelocityEngine.FILE_RESOURCE_LOADER_PATH, "d://");
Velocity.init(p);
...
Velocity.getTemplate("tree.vm");
 
 
方式三：使用文本文件，如：velocity.properties，配置如下：
#encoding
input.encoding=UTF-8
output.encoding=UTF-8
contentType=text/html;charset=UTF-8
不要指定loader.

 
再利用如下方式进行加载
Properties p = new Properties();
p.load(this.getClass().getResourceAsStream("/velocity.properties"));
Velocity.init(p);
...
Velocity.getTemplate(templateFile);

package com.study.volicity;

import java.io.IOException;
import java.io.StringWriter;
import java.util.Properties;

import org.apache.velocity.app.Velocity;
import org.apache.velocity.Template;
import org.apache.velocity.VelocityContext;

public class Test {
	

	public static void main(String args[]) throws IOException {

		Properties pros = new Properties();
		pros.load(Test.class.getClassLoader().getResourceAsStream("velocity.properties"));
		Velocity.init(pros);
		VelocityContext context = new VelocityContext();

		context.put("name", "Velocity");
		context.put("project", "Jakarta");

		/* lets render a template 相对项目路径 */
		Template template = Velocity.getTemplate("/view/header.vm");

		StringWriter writer = new StringWriter();

		/* lets make our own string to render */
		template.merge(context, writer);
		System.out.println(writer);
	}

}