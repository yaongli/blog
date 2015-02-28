## Using prose.io to create A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

```java

import java.io.IOException;
import java.util.Random;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.SequenceFileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.map.InverseMapper;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;


public class LogParser {

	public static void main(String[] args) throws IOException,
			InterruptedException, ClassNotFoundException {

		Path inputPath = new Path(args[0]);
		Path tempDir = new Path("wordcount-temp-" + Integer.toString(new Random().nextInt(Integer.MAX_VALUE)));
		Path outputDir = new Path(args[1]);

		// Create configuration
		Configuration conf = new Configuration(true);

		// Create job
		Job job = new Job(conf, "WordCount");
		job.setJarByClass(LogParser.class);

		// Setup MapReduce
		job.setMapperClass(LogParserMapper.class);
		job.setReducerClass(LogParserReducer.class);
		//job.setSortComparatorClass(LogParserSorter.class);
		job.setNumReduceTasks(1);

		// Specify key / value
		job.setOutputKeyClass(Text.class);
	    job.setOutputValueClass(IntWritable.class);
	    job.setMapOutputKeyClass(Text.class);
	    job.setMapOutputValueClass(IntWritable.class);
	    
		// Input
		FileInputFormat.addInputPath(job, inputPath);
		job.setInputFormatClass(TextInputFormat.class);

		// Output
		FileOutputFormat.setOutputPath(job, tempDir);
		job.setOutputFormatClass(SequenceFileOutputFormat.class);

		// Delete output if exists
		FileSystem hdfs = FileSystem.get(conf);
		if (hdfs.exists(outputDir))
			hdfs.delete(outputDir, true);
		if (hdfs.exists(tempDir))
			hdfs.delete(tempDir, true);

		// Execute job
		int code = job.waitForCompletion(true) ? 0 : 1;
		
		if(1 == code){
			System.exit(code);
			return;
		}
		
		Configuration sortConf = new Configuration(true);

		// Create job
		Job sortJob = new Job(sortConf, "Sort Count");
		sortJob.setJarByClass(LogParser.class);
		// Setup MapReduce
		sortJob.setMapperClass(InverseMapper.class);
		sortJob.setSortComparatorClass(LogParserSorter.class);
		sortJob.setNumReduceTasks(1);
		
		// Input
		FileInputFormat.addInputPath(sortJob, tempDir);
		sortJob.setInputFormatClass(SequenceFileInputFormat.class);

		// Output
		FileOutputFormat.setOutputPath(sortJob, outputDir);
		sortJob.setOutputFormatClass(TextOutputFormat.class);
		
        sortJob.setOutputKeyClass(IntWritable.class);
        sortJob.setOutputValueClass(Text.class);
        
        
        code = sortJob.waitForCompletion(true) ? 0 : 1;
        
        if (hdfs.exists(tempDir))
			hdfs.delete(tempDir, true);
        System.exit(code);
	}

}

```


Bash Script
```bash
#
# Helper to complain.
#
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $*"
}

warn() {
    log "${PROGNAME}: $*"
}

#
# Helper to puke.
#
die() {
    warn $*
    exit 1
}

#
# run command
#
run_command() {
    echo ""
    echo "Command:"
    echo "-----------------------------------------------------------------"
    echo "$@"
    eval $@
    if [ $? -ne 0 ];then
        die "[ERROR] Failed to exec command: $@"
    fi
    echo "-----------------------------------------------------------------"
    echo ""
}

tomcat_instances_trigger() {
    log "tomcat instances trigger"
    for trigger in ${TOMCAT_TRIGGER_URL[*]};
    do
        run_command "curl -k ${trigger}${DATABASE}"
    done
}
```
