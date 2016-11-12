```
package org.apache.myhadoop.recommend;

import java.util.HashMap;
import java.util.Map;
import java.util.regex.Pattern;

import org.apache.hadoop.mapred.JobConf;

public class Recommend {
	public static final String HDFS = "hdfs://127.0.0.1:9000";
	public static final Pattern DELIMITER = Pattern.compile("[\t,]");
	
	public static void main(String[] args) throws Exception {
	    Map<String, String> path = new HashMap<String, String>();
	    path.put("data", "/usr/lib/hadoop/hello.csv");
	    path.put("Step1Input", HDFS + "/user/hadoop");
	    path.put("Step1Output", path.get("Step1Input") + "/step1");
	    path.put("Step2Input", path.get("Step1Output"));
	    path.put("Step2Output", path.get("Step1Input") + "/step2");
	    path.put("Step3Input1", path.get("Step1Output"));
	    path.put("Step3Output1", path.get("Step1Input") + "/step3_1");
	    path.put("Step3Input2", path.get("Step2Output"));
	    path.put("Step3Output2", path.get("Step1Input") + "/step3_2");
	    
	    /*path.put("Step4Input1", path.get("Step3Output1"));
	    path.put("Step4Input2", path.get("Step3Output2"));
	    path.put("Step4Output", path.get("Step1Input") + "/step4");*/
	
	    path.put("Step4Input1", path.get("Step3Output1"));
        path.put("Step4Input2", path.get("Step3Output2"));
        path.put("Step4Output", path.get("Step1Input") + "/step4");
        
        path.put("Step5Input", path.get("Step4Output"));
        path.put("Step5Output", path.get("Step1Input") + "/step5");  
	    Step1.run(path);
	    Step2.run(path);
	    Step3.run1(path);
	    Step3.run2(path);
	    Step4_Update.run(path);
        Step4_Update2.run(path);
	    System.exit(0);
	}
	
	public static JobConf config() {
	    JobConf conf = new JobConf(Recommend.class);
	    conf.setJobName("Recommend");
	    conf.addResource("classpath:core-site.xml");
	    conf.addResource("classpath:hdfs-site.xml");
	    conf.addResource("classpath:mapred-site.xml");
	    //conf.set("io.sort.mb", "1024");
	    return conf;
	}
}
```
```
package org.apache.myhadoop.recommend;

import java.io.IOException;
import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.mapred.JobConf;

public class HdfsDAO {
	private static final String HDFS = "hdfs://127.0.0.1:9000/";

    public HdfsDAO(Configuration conf) {
        this(HDFS, conf);
    }

    public HdfsDAO(String hdfs, Configuration conf) {
        this.hdfsPath = hdfs;
        this.conf = conf;
    }

    private String hdfsPath;
    private Configuration conf;

    public static void main(String[] args) throws IOException {
        JobConf conf = config();
        HdfsDAO hdfs = new HdfsDAO(conf);
        hdfs.copyFile("datafile/item.csv", "/tmp/new");
        hdfs.ls("/tmp/new");
    }        

    public static JobConf config(){
        JobConf conf = new JobConf(HdfsDAO.class);
        conf.setJobName("HdfsDAO");
        conf.addResource("classpath:core-site.xml");
        conf.addResource("classpath:hdfs-site.xml");
        conf.addResource("classpath:mapred-site.xml");
        return conf;
    }

    public void mkdirs(String folder) throws IOException {
        Path path = new Path(folder);
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        if (!fs.exists(path)) {
            fs.mkdirs(path);
            System.out.println("Create: " + folder);
        }
        fs.close();
    }
    public void rmr(String folder) throws IOException {
        Path path = new Path(folder);
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        fs.deleteOnExit(path);
        System.out.println("Delete: " + folder);
        fs.close();
    }

    public void ls(String folder) throws IOException {
        Path path = new Path(folder);
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        FileStatus[] list = fs.listStatus(path);
        System.out.println("ls: " + folder);
        System.out.println("==========================================================");
        for (FileStatus f : list) {
            System.out.printf("name: %s, folder: %s, size: %d\n", f.getPath(), f.isDir(), f.getLen());
        }
        System.out.println("==========================================================");
        fs.close();
    }

    public void createFile(String file, String content) throws IOException {
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        byte[] buff = content.getBytes();
        FSDataOutputStream os = null;
        try {
            os = fs.create(new Path(file));
            os.write(buff, 0, buff.length);
            System.out.println("Create: " + file);
        } finally {
            if (os != null)
                os.close();
        }
        fs.close();
    }
    public void copyFile(String local, String remote) throws IOException {
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        fs.copyFromLocalFile(new Path(local), new Path(remote));
        System.out.println("copy from: " + local + " to " + remote);
        fs.close();
    }

    public void download(String remote, String local) throws IOException {
        Path path = new Path(remote);
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        fs.copyToLocalFile(path, new Path(local));
        System.out.println("download: from" + remote + " to " + local);
        fs.close();
    }

    public void cat(String remoteFile) throws IOException {
        Path path = new Path(remoteFile);
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        FSDataInputStream fsdis = null;
        System.out.println("cat: " + remoteFile);
        try {  
            fsdis =fs.open(path);
            IOUtils.copyBytes(fsdis, System.out, 4096, false);  
          } finally {  
            IOUtils.closeStream(fsdis);
            fs.close();
          }
    }
}
```
```
package org.apache.myhadoop.recommend;

import java.io.IOException;
import java.util.Iterator;
import java.util.Map;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.RunningJob;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;

public class Step1 {
	public static class Step1_ToItemPreMapper extends MapReduceBase implements Mapper<Object, Text, IntWritable, Text> {
	    private final static IntWritable k = new IntWritable();
	    private final static Text v = new Text();
	
	    public void map(Object key, Text value, OutputCollector<IntWritable, Text> output, Reporter reporter) throws IOException {
	        String[] tokens = Recommend.DELIMITER.split(value.toString());
	        int userID = Integer.parseInt(tokens[0]);
	        String itemID = tokens[1];
	        String pref = tokens[2];
	        k.set(userID);
	        v.set(itemID + ":" + pref);
	        output.collect(k, v);
	    }
	}
	//public static class Step1_ToUserVectorReducer extends MapReduceBase implements Reducer<IntWritable, Text,IntWritable,Text> {
	public static class Step1_ToUserVectorReducer extends MapReduceBase implements Reducer<IntWritable,Text,IntWritable,Text> {
	    private final static Text v = new Text();
	   
	    public void reduce(IntWritable key, Iterator<Text> values, OutputCollector<IntWritable, Text> output, Reporter reporter) throws IOException {
	        StringBuilder sb = new StringBuilder();
	        while (values.hasNext()) {
	            sb.append("," + values.next());
	        }
	        v.set(sb.toString().replaceFirst(",", ""));
	        output.collect(key, v);
	    }

		/*public void reduce(Text key, Iterator<IntWritable> arg1,
				OutputCollector<Text, IntWritable> arg2, Reporter arg3)
				throws IOException {
			// TODO Auto-generated method stub
			
		}*/

		
	}
	public static void run(Map<String, String> path) throws IOException {
        JobConf conf = Recommend.config();

        String input = path.get("Step1Input");
        String output = path.get("Step1Output");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(input);
        hdfs.mkdirs(input);
        hdfs.copyFile(path.get("data"), input);

        //conf.setMapOutputKeyClass(IntWritable.class);
        //conf.setMapOutputKeyClass(IntWritable.class);
        //conf.setMapOutputValueClass(Text.class);

        conf.setOutputKeyClass(IntWritable.class);
        conf.setOutputValueClass(Text.class);
        

        conf.setMapperClass(Step1_ToItemPreMapper.class);
        conf.setCombinerClass(Step1_ToUserVectorReducer.class);
        conf.setReducerClass(Step1_ToUserVectorReducer.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(input));
        FileOutputFormat.setOutputPath(conf, new Path(output));

        RunningJob job = JobClient.runJob(conf);
        while (!job.isComplete()) {
            job.waitForCompletion();
        }
    }

}
```
```
package org.apache.myhadoop.recommend;

import java.io.IOException;
import java.util.Iterator;
import java.util.Map;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.RunningJob;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;

public class Step2 {
	public static class Step2_UserVectorToCooccurrenceMapper extends MapReduceBase implements Mapper<LongWritable, Text, Text, IntWritable> {
	    private final static Text k = new Text();
	    private final static IntWritable v = new IntWritable(1);
	
	    
	    public void map(LongWritable key, Text values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
	        String[] tokens = Recommend.DELIMITER.split(values.toString());
	        for (int i = 1; i < tokens.length; i++) {
	            String itemID = tokens[i].split(":")[0];
	            for (int j = 1; j < tokens.length; j++) {
	                String itemID2 = tokens[j].split(":")[0];
	                k.set(itemID + ":" + itemID2);
	                output.collect(k, v);
	            }
	        }
	    }
	}
	public static class Step2_UserVectorToConoccurrenceReducer extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

 
        public void reduce(Text key, Iterator<IntWritable> values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
            int sum = 0;
            while (values.hasNext()) {
                sum += ((IntWritable) values.next()).get();
            }
            result.set(sum);
            output.collect(key, result);
        }


    }

	public static void run(Map<String, String> path) throws IOException {
        JobConf conf = Recommend.config();

        String input = path.get("Step2Input");
        String output = path.get("Step2Output");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(output);

        conf.setOutputKeyClass(Text.class);
        conf.setOutputValueClass(IntWritable.class);

        conf.setMapperClass(Step2_UserVectorToCooccurrenceMapper.class);
        conf.setCombinerClass(Step2_UserVectorToConoccurrenceReducer.class);
        conf.setReducerClass(Step2_UserVectorToConoccurrenceReducer.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(input));
        FileOutputFormat.setOutputPath(conf, new Path(output));

        RunningJob job = JobClient.runJob(conf);
        while (!job.isComplete()) {
            job.waitForCompletion();
        }
    }
	
}
```
```
package org.apache.myhadoop.recommend;

import java.io.IOException;
import java.util.Map;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.RunningJob;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;

public class Step3 {

	public static class Step31_UserVectorSplitterMapper extends MapReduceBase implements Mapper<LongWritable, Text, IntWritable, Text> {
        private final static IntWritable k = new IntWritable();
        private final static Text v = new Text();

       
        public void map(LongWritable key, Text values, OutputCollector<IntWritable, Text> output, Reporter reporter) throws IOException {
            String[] tokens = Recommend.DELIMITER.split(values.toString());
            for (int i = 1; i < tokens.length; i++) {
                String[] vector = tokens[i].split(":");
                int itemID = Integer.parseInt(vector[0]);
                String pref = vector[1];

                k.set(itemID);
                v.set(tokens[0] + ":" + pref);
                output.collect(k, v);
            }
        }
    }
	public static void run1(Map<String, String> path) throws IOException {
        JobConf conf = Recommend.config();

        String input = path.get("Step3Input1");
        String output = path.get("Step3Output1");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(output);

        conf.setOutputKeyClass(IntWritable.class);
        conf.setOutputValueClass(Text.class);

        conf.setMapperClass(Step31_UserVectorSplitterMapper.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(input));
        FileOutputFormat.setOutputPath(conf, new Path(output));

        RunningJob job = JobClient.runJob(conf);
        while (!job.isComplete()) {
            job.waitForCompletion();
        }
    }
	public static class Step32_CooccurrenceColumnWrapperMapper extends MapReduceBase implements Mapper<LongWritable, Text, Text, IntWritable> {
        private final static Text k = new Text();
        private final static IntWritable v = new IntWritable();

        
        public void map(LongWritable key, Text values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
            String[] tokens = Recommend.DELIMITER.split(values.toString());
            k.set(tokens[0]);
            v.set(Integer.parseInt(tokens[1]));
            output.collect(k, v);
        }
    }
	public static void run2(Map<String, String> path) throws IOException {
        JobConf conf = Recommend.config();

        String input = path.get("Step3Input2");
        String output = path.get("Step3Output2");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(output);

        conf.setOutputKeyClass(Text.class);
        conf.setOutputValueClass(IntWritable.class);

        conf.setMapperClass(Step32_CooccurrenceColumnWrapperMapper.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(input));
        FileOutputFormat.setOutputPath(conf, new Path(output));

        RunningJob job = JobClient.runJob(conf);
        while (!job.isComplete()) {
            job.waitForCompletion();
        }
    }

}
```
```
package org.apache.myhadoop.recommend;


import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class Step4_Update {

    public static class Step4_PartialMultiplyMapper extends Mapper<LongWritable, Text, Text, Text> {

        private String flag;// A同现矩阵 or B评分矩阵

        @Override
        protected void setup(Context context) throws IOException, InterruptedException {
            FileSplit split = (FileSplit) context.getInputSplit();
            flag = split.getPath().getParent().getName();// 判断读的数据集

            // System.out.println(flag);
        }

        @Override
        public void map(LongWritable key, Text values, Context context) throws IOException, InterruptedException {
            String[] tokens = Recommend.DELIMITER.split(values.toString());

            if (flag.equals("step3_2")) {// 同现矩阵
                String[] v1 = tokens[0].split(":");
                String itemID1 = v1[0];
                String itemID2 = v1[1];
                String num = tokens[1];

                Text k = new Text(itemID1);
                Text v = new Text("A:" + itemID2 + "," + num);

                context.write(k, v);
                // System.out.println(k.toString() + "  " + v.toString());

            } else if (flag.equals("step3_1")) {// 评分矩阵
                String[] v2 = tokens[1].split(":");
                String itemID = tokens[0];
                String userID = v2[0];
                String pref = v2[1];

                Text k = new Text(itemID);
                Text v = new Text("B:" + userID + "," + pref);

                context.write(k, v);
                // System.out.println(k.toString() + "  " + v.toString());
            }
        }

    }

    public static class Step4_AggregateReducer extends Reducer<Text, Text, Text, Text> {

        @Override
        public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            System.out.println(key.toString() + ":");

            Map<String, String> mapA = new HashMap<String, String>();
            Map<String, String> mapB = new HashMap<String, String>();

            for (Text line : values) {
                String val = line.toString();
                System.out.println(val);

                if (val.startsWith("A:")) {
                    String[] kv = Recommend.DELIMITER.split(val.substring(2));
                    mapA.put(kv[0], kv[1]);

                } else if (val.startsWith("B:")) {
                    String[] kv = Recommend.DELIMITER.split(val.substring(2));
                    mapB.put(kv[0], kv[1]);

                }
            }

            double result = 0;
            Iterator<String> iter = mapA.keySet().iterator();
            while (iter.hasNext()) {
                String mapk = iter.next();// itemID

                int num = Integer.parseInt(mapA.get(mapk));
                Iterator<String> iterb = mapB.keySet().iterator();
                while (iterb.hasNext()) {
                    String mapkb = iterb.next();// userID
                    double pref = Double.parseDouble(mapB.get(mapkb));
                    result = num * pref;// 矩阵乘法相乘计算

                    Text k = new Text(mapkb);
                    Text v = new Text(mapk + "," + result);
                    context.write(k, v);
                    System.out.println(k.toString() + "  " + v.toString());
                }
            }
        }
    }

    public static void run(Map<String, String> path) throws IOException, InterruptedException, ClassNotFoundException {
        JobConf conf = Recommend.config();

        String input1 = path.get("Step4Input1");
        String input2 = path.get("Step4Input2");
        String output = path.get("Step4Output");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(output);

        Job job = new Job(conf);
        job.setJarByClass(Step4_Update.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        job.setMapperClass(Step4_Update.Step4_PartialMultiplyMapper.class);
        job.setReducerClass(Step4_Update.Step4_AggregateReducer.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.setInputPaths(job, new Path(input1), new Path(input2));
        FileOutputFormat.setOutputPath(job, new Path(output));

        job.waitForCompletion(true);
    }

}
```
```
package org.apache.myhadoop.recommend;


import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class Step4_Update2 {

    public static class Step4_RecommendMapper extends Mapper<LongWritable, Text, Text, Text> {

        @Override
        public void map(LongWritable key, Text values, Context context) throws IOException, InterruptedException {
            String[] tokens = Recommend.DELIMITER.split(values.toString());
            Text k = new Text(tokens[0]);
            Text v = new Text(tokens[1]+","+tokens[2]);
            context.write(k, v);
        }
    }

    public static class Step4_RecommendReducer extends Reducer<Text, Text, Text, Text> {
        
        @Override
        public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            System.out.println(key.toString() + ":");
            Map<String, Double> map = new HashMap<String, Double>();// 结果
            
            for (Text line : values) {
                System.out.println(line.toString());
                String[] tokens = Recommend.DELIMITER.split(line.toString());
                String itemID = tokens[0];
                Double score = Double.parseDouble(tokens[1]);
                
                 if (map.containsKey(itemID)) {
                     map.put(itemID, map.get(itemID) + score);// 矩阵乘法求和计算
                 } else {
                     map.put(itemID, score);
                 }
            }
            
            Iterator<String> iter = map.keySet().iterator();
            while (iter.hasNext()) {
                String itemID = iter.next();
                double score = map.get(itemID);
                Text v = new Text(itemID + "," + score);
                context.write(key, v);
            }
        }
    }

    public static void run(Map<String, String> path) throws IOException, InterruptedException, ClassNotFoundException {
        JobConf conf = Recommend.config();

        String input = path.get("Step5Input");
        String output = path.get("Step5Output");

        HdfsDAO hdfs = new HdfsDAO(Recommend.HDFS, conf);
        hdfs.rmr(output);

        Job job = new Job(conf);
        job.setJarByClass(Step4_Update2.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        job.setMapperClass(Step4_Update2.Step4_RecommendMapper.class);
        job.setReducerClass(Step4_Update2.Step4_RecommendReducer.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.setInputPaths(job, new Path(input));
        FileOutputFormat.setOutputPath(job, new Path(output));

        job.waitForCompletion(true);
    }

}
```
