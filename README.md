# Stormdemo
创建IntelliJ IDEA项目
选择maven



填写groupId和artifactId



填写项目名称和项目路径



修改pom.xml文件
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zhyoulun</groupId>
    <artifactId>storm_study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>0.9.7</version>
        </dependency>
    </dependencies>

</project>

编写Topology、Spout、Bolt文件
模型图为



代码位置



MainTopology.java文件

import backtype.storm.Config;
import backtype.storm.LocalCluster;
import backtype.storm.topology.TopologyBuilder;

public class MainTopology {
    public void runLocal(int waitSeconds) {
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout("wordSpout", new WordSpout(), 1);
        builder.setBolt("countBolt", new CountBolt(), 1).shuffleGrouping("wordSpout");

        Config config = new Config();
        LocalCluster cluster = new LocalCluster();
        cluster.submitTopology("word_count", config, builder.createTopology());

        try {
            Thread.sleep(waitSeconds * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        cluster.killTopology("word_count");
        cluster.shutdown();
    }

    public static void main(String[] args) {
        MainTopology topology = new MainTopology();
        topology.runLocal(60);
    }
}

WordSpout.java文件

import backtype.storm.spout.SpoutOutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichSpout;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Values;

import java.util.Map;


public class WordSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private String[] words = {"hello","world","storm","study"};//单词池
    private int index = 0;

    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
        this.collector = spoutOutputCollector;
    }

    public void nextTuple() {
        this.collector.emit(new Values(this.words[index]));
        index++;
        if(index>=words.length){
            index = 0;
        }

        //等待500ms
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word"));
    }
}

CountBolt.java文件

import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Tuple;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;


public class CountBolt extends BaseRichBolt {
    private HashMap<String, Integer> wordMap = new HashMap<String, Integer>();

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {

    }

    public void execute(Tuple tuple) {
        //从tuple中读取单词
        String word = tuple.getStringByField("word");

        //计数
        int num;
        if (wordMap.containsKey(word)) {
            num = wordMap.get(word);
        } else {
            num = 0;
        }
        wordMap.put(word, 1 + num);

        //输出展示
        Set<String> keys = wordMap.keySet();
        for (String key : keys) {
            System.out.print(key + ":" + wordMap.get(key) + ",");
        }
        System.out.println();
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }
}

运行结果
...
6709 [Thread-11-wordSpout] INFO  backtype.storm.daemon.executor - Activating spout wordSpout:(3)
hello:1,
world:1,hello:1,
world:1,storm:1,hello:1,
study:1,world:1,storm:1,hello:1,
study:1,world:1,storm:1,hello:2,
study:1,world:2,storm:1,hello:2,
study:1,world:2,storm:2,hello:2,
study:2,world:2,storm:2,hello:2,
study:2,world:2,storm:2,hello:3,
study:2,world:3,storm:2,hello:3,
study:2,world:3,storm:3,hello:3,
...
study:27,world:27,storm:27,hello:28,
study:27,world:28,storm:27,hello:28,
study:27,world:28,storm:28,hello:28,
study:28,world:28,storm:28,hello:28,
study:28,world:28,storm:28,hello:29,
study:28,world:29,storm:28,hello:29,
study:28,world:29,storm:29,hello:29,
study:29,world:29,storm:29,hello:29,
64515 [main] INFO  backtype.storm.daemon.nimbus - Delaying event :remove for 30 secs for word_count-1-1511504981
...

java.lang.NoClassDefFoundError: backtype/storm/topology/IRichSpout
如果pom.xml中的dependencies部分如下所示：

<dependencies>
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>0.9.7</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

将会报出这个错误，详细内容如下所示：

java.lang.NoClassDefFoundError: backtype/storm/topology/IRichSpout
    at java.lang.Class.getDeclaredMethods0(Native Method)
    at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
    at java.lang.Class.privateGetMethodRecursive(Class.java:3048)
    at java.lang.Class.getMethod0(Class.java:3018)
    at java.lang.Class.getMethod(Class.java:1784)
    at sun.launcher.LauncherHelper.validateMainClass(LauncherHelper.java:544)
    at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:526)
Caused by: java.lang.ClassNotFoundException: backtype.storm.topology.IRichSpout
    at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:335)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ... 7 more
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" 

解决这个问题需要把<scope>provided</scope>删掉。指定该属性的意思是，运行环境已经提供了相应的依赖，该依赖不会被打进jar包中。而在intellij中运行本地模式时，需要删掉这个属性，否则会找不到类。
