This is the Java Development Kit. Let's build it by parts:
JVM -> Java Virtual Machine: The program that actually runs your compiled code. Your code doesn't really touch the cpu, the JVM does.
JRE -> JVM + Standard library: Set of prewritten functions and libraries that are enough to run a program (i.e. java.util, java.lang...).
JDK -> JRE + developer tools:
	Inside these dev tools we could have:
		- javac -> Is what compiles your code from .java to .class bytecode
		- java -> It launches the JVM (and tells it what to run)
		- jar -> packs .class files + manifest + resources (like a .zip but with java-specific layout) every dependency Maven downloads is a Jar (also when executing mvn package -> it's producing a jar from your project)
		- javadoc -> Reads the javadoc and generates HTML API documentation.
		- jshell -> an interactive REPL, type a Java expression and get the result immediately.
		- debugger interface -> The JDK ships both the protocol (JDWP) and the API (JDI) that lets external tools pause the JVM, inspect variables and step through code. The vscode java debugger is a client of that. You never use it directly, only through the IDE.