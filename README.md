# 📚 Reading:  Amazon Simple Storage Service (S3)

<div id="git-data-element" data-org="learn-co-curriculum" data-repo="dsc-aws-s3"></div>
<header class="fis-header" style="visibility: hidden;"><a class="fis-git-link" href="https://github.com/learn-co-curriculum/dsc-aws-s3" target="_blank"><img id="repo-img" title="Open GitHub Repo" alt="GitHub Repo"></a><a class="fis-git-link" href="https://github.com/learn-co-curriculum/dsc-aws-s3/issues/new" target="_blank"><img id="issue-img" title="Create New Issue" alt="Create New Issue"></a></header>
<h2>Introduction</h2>
<p>Amazon Simple Storage Service (S3) is one of the flagship services from AWS. In this lesson we'll discuss how you might use it as a data scientist!</p>
<h2>Objectives</h2>
<p>You will be able to:</p>
<ul>
<li>Describe use cases for S3 buckets in data science</li>
<li>Create S3 buckets and upload data</li>
<li>Access data in S3 buckets with Python code</li>
</ul>
<h2>S3 Buckets in Data Science</h2>
<h3>Limitations of GitHub</h3>
<p>At this point in the program, you might have encountered an error message like this more than once:</p>
<pre style="color: red;">remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com
remote: error: Trace: 08740bd2fb02f980041be67b73e715a9
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File is 218.83 MB; this exceeds GitHub's file size limit of 100.00 MB
! [remote rejected] master -&gt; master (pre-receive hook declined)
error: failed to push some refs
</pre>
<p>This happens when you try to push a file that exceeds GitHub's <a href="https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github">file size limits</a>. The file might contain data in CSV or JSON format, or maybe a pickled ML model.</p>
<p>The short-term workaround is to use <code>.gitignore</code> so that the file or files just stay on your computer and are not pushed to GitHub. But that has some limitations:</p>
<ul>
<li>If you want someone else to be able to <strong>reproduce</strong> your project and one or more files are too big for GitHub, how do they get the file(s)? You would need to provide very detailed instructions to make this possible</li>
<li>If you want to <strong>train your model in the cloud</strong> and your data is too big for GitHub, how will your code access the data?</li>
<li>If you want to <strong>productionize your model</strong> and your pickled model is too big for GitHub, how will your code access the model?</li>
</ul>
<p>This problem is also not limited to GitHub. Some cloud notebooks do not support standard file systems at all, and some deployment approaches have file size limitations that are much more restrictive than GitHub's!</p>
<p>Fortunately, while these tools might not be optimized for larger files, S3 buckets are a great alternative.</p>
<h3>Recommended S3 Setup for Data Science</h3>
<p>For your projects, let's assume you are:</p>
<ul>
<li>Working from a Jupyter notebook locally</li>
<li>Not concerned about access or keeping data private</li>
<li>Not needing to dynamically upload data (e.g. allow users to upload photos)</li>
</ul>
<p>Therefore we recommend that you follow this S3 setup:</p>
<ol>
<li>Set up an S3 bucket where objects are all publicly readable</li>
<li>When you encounter or create a file that is too big for GitHub, <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html">upload files to S3</a>
<ol>
<li>For files below 160 GB, use the AWS Console (web browser interface)</li>
<li>For files between 160 GB and 5 TB, use <code>boto3</code> (the AWS Python SDK)</li>
</ol>
</li>
<li>Use <code>boto3</code> in your Python code to access data in your bucket</li>
</ol>
<p>Now we will go over all of these steps!</p>
<h2>Creating and Configuring S3 Buckets</h2>
<p>In some software applications, S3 buckets are created dynamically based on application context. For data science, projects don't typically need to be quite so dynamic, so we can create our S3 buckets using the more-intuitive AWS console interface rather than using code.</p>
<p>Go to the <strong>S3 management console</strong>. This can be accessed by searching for "s3" in the <a href="https://aws.amazon.com/console/">AWS Management Console</a> or by going directly to <a href="https://s3.console.aws.amazon.com/s3/home">this link</a>.</p>
<p>Click on <strong>"Create Bucket"</strong>.</p>
<p>Give your bucket a <strong>unique name</strong>:</p>
<ul>
<li>If you're working on a specific project, try giving the bucket a name that relates to the project</li>
<li>DO NOT include any private/secret information in the bucket name</li>
<li>If someone else has already used a bucket name, you will not be able to use it
<ul>
<li>If you're feeling stuck brainstorming a name, try adding today's date to it. That will be less likely to already be in use</li>
</ul>
</li>
</ul>
<p>Scroll down and <strong>un-check "Block all public access"</strong>. AWS assumes that you want your data to be private, but we are moving forward with the assumption that public data access is not a problem. You will also need to check the box next to <strong>I acknowledge that the current settings might result in this bucket and the objects within becoming public</strong>.</p>
<p>Scroll the rest of the way down and click <strong>Create Bucket</strong>.</p>
<p>Now you should have a bucket set up where objects can be public!</p>
<h2>Uploading Data to S3 Buckets</h2>
<h3>AWS Console Approach</h3>
<p>For files below 160 GB, you can use the AWS Management Console. Go to the <strong>S3 management console</strong> and click on the name of your bucket.</p>
<p>By default, you should see the "Objects" tab. Click on <strong>Upload</strong>. Click <strong>Add Files</strong> and use the file picker to select a file or multiple files on your computer.</p>
<p>Scroll down to <strong>Permissions</strong> and click to expand. Under "Predefined ACLs", click <strong>Grant public-read access</strong> and then check the box next to <strong>I understand the risk of granting public-read access to the specified objects</strong>.</p>
<p>Scroll to the bottom and click <strong>Upload</strong>. You will be taken to an upload status page while the file is being uploaded, then you can click <strong>Close</strong>.</p>
<h3>Checking for Successful Upload and Configuration</h3>
<p>Now your data should be publicly hosted on S3! Try clicking on the file name in the "Objects" tab to see all of the information about it.</p>
<p>To test whether the file permissions were successfully set to be public, you can click on either the "Open" button in the upper right, or the "Object URL" link inside the "Object Overview" panel.</p>
<h4>Successful Upload</h4>
<p>If your file is downloaded, that means the settings are correct. (If it is a very large file, feel free to stop the download to save time and space -- you already have the file on your computer!)</p>
<h4>Unsuccessful Upload</h4>
<p>If you see a message like this:</p>
<blockquote>
<p>This XML file does not appear to have any style information associated with it. The document tree is shown below.</p>
</blockquote>
<div class="highlight">
<pre class="highlight plaintext"><code>&lt;Error&gt;
&lt;Code&gt;AccessDenied&lt;/Code&gt;
&lt;Message&gt;Access Denied&lt;/Message&gt;
&lt;RequestId&gt;XXXXXXXXXXXXXXXX&lt;/RequestId&gt;
&lt;HostId&gt;XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX&lt;/HostId&gt;
&lt;/Error&gt;
</code></pre>
</div>
<p>That means that you did not configure the permissions correctly. Go back through and make sure that:</p>
<ul>
<li>The bucket settings are configured so that objects can be public
<ul>
<li>If you look at the bucket from the <a href="https://s3.console.aws.amazon.com/s3/home">S3 management console</a>, you should see the text "Objects can be public" in the "Access" column</li>
<li>If you do not see that, click on the bucket, go to the <strong>Permissions</strong> tab, scroll down to <strong>Block public access (bucket settings)</strong>, and click <strong>Edit</strong>. Un-check <strong>Block all public access</strong>, check <strong>I acknowledge that the current settings might result in this bucket and the objects within becoming public</strong>, and click <strong>Save Changes</strong></li>
</ul>
</li>
<li>The object settings are configured so that the public can access them
<ul>
<li>If you click on the bucket name, then click on the name of the file (e.g. <code>test.csv</code>), then go to the <strong>Permissions</strong> tab, you should see "Read" in the "Object" column next to "Everyone (public access)"</li>
<li>If you do not see that, click on <strong>Edit</strong> next to "Access control list (ACL)". Locate the row that says "Everyone (public access)" and the column that says "Objects", and click the checkbox next to <strong>Read</strong>. Scroll down and click the checkbox next to <strong>I understand the effects of these changes on this object</strong>, then click <strong>Save changes</strong></li>
</ul>
</li>
</ul>
<h3>Python Approach</h3>
<p>For files from 160 GB to 5 TB, you'll need a more sophisticated approach. Check out <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html">this documentation</a> for an overview of the multi-part upload process.</p>
<p>Fortunately you can use <code>boto3</code>, the same package we'll use in the later examples in this lesson, in order to achieve this. First, use <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration">this documentation</a> to configure a file on your computer to give <code>boto3</code> the credentials it needs to upload. Then follow the documentation <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.create_multipart_upload">here</a> to create the actual upload.</p>
<h2>Accessing Data in S3 Buckets</h2>
<p>As demonstrated above, one way to access the data in your S3 bucket is simply to navigate the the relevant web address and download the file. However, our main goal is to make the data accessible in <strong>Python code</strong>, not using a web browser. To achieve that, we'll use the <code>boto3</code> library!</p>
<p><img src="https://curriculum-content.s3.amazonaws.com/data-science/images/boto3.png" alt="boto3"></p>
<p>Boto 3 is a library that allows Python developers to access many different Amazon web services, not just S3. You can find the full list <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/index.html">here</a>. But we'll focus on using S3 with <code>boto3</code> because that is one of the most common use cases for data scientists.</p>
<p>(The name "boto" is a type of dolphin that is native to the Amazon river. <a href="https://github.com/boto/boto3/issues/1023#issuecomment-287127647">According to its developer</a>, "I wanted something short, unusual, and with at least some kind of connection to Amazon".)</p>
<h3>Installing <code>boto3</code></h3>
<p>This library is not part of <code>learn-env</code> as of this writing. If you are working on a project, we recommend activating that project <code>conda</code> environment. Or if you're just wanting to learn about <code>boto3</code>, you can un-comment this line of code to install it in your current environment:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="c1"># !conda install boto3 -y
</span></code></pre>
</div>
<p>Make sure this cell runs successfully before proceeding:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="nn">boto3</span>
<span class="n">s3</span> <span class="o">=</span> <span class="n">boto3</span><span class="p">.</span><span class="n">resource</span><span class="p">(</span><span class="s">"s3"</span><span class="p">)</span>
</code></pre>
</div>
<h3>Connecting to an Example S3 Bucket</h3>
<p>For the purposes of this curriculum, we have loaded some example files into an S3 bucket for you! Let's go through those examples, starting with a text file.</p>
<h4>Text File</h4>
<p>We'll instantiate an instance of the <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html?highlight=s3.object#S3.Object"><code>Object</code> class</a> by passing in those string values. The first one, <code>bucket_name</code>, is the name of the bucket. The second, <code>key</code>, is the name of the object (file) inside that bucket.</p>
<div class="highlight">
<pre class="highlight python"><code><span class="n">txt_obj</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="n">Object</span><span class="p">(</span><span class="s">"curriculum-content"</span><span class="p">,</span> <span class="s">"data-science/data/zen_of_python.txt"</span><span class="p">)</span>
<span class="n">txt_obj</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight plaintext"><code>s3.Object(bucket_name='curriculum-content', key='data-science/data/zen_of_python.txt')
</code></pre>
</div>
<p>This object is used kind of like a request in the <code>requests</code> library. You call the <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html?highlight=s3.object#S3.Object.get"><code>get</code> method</a> to initiate an HTTP <code>GET</code> request.</p>
<div class="highlight">
<pre class="highlight python"><code><span class="n">txt_resp</span> <span class="o">=</span> <span class="n">txt_obj</span><span class="p">.</span><span class="n">get</span><span class="p">()</span>
<span class="n">txt_resp</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight plaintext"><code>{'ResponseMetadata': {'RequestId': 'JR1N0QAM82W3ETVW',
  'HostId': 'J2J6MiFzynwLLLlOXBSTgYyFOPcDpoofyc165VnBbB+nNEH2cbovo7p4+MGMYXiT0mfD8acVZ2o=',
  'HTTPStatusCode': 200,
  'HTTPHeaders': {'x-amz-id-2': 'J2J6MiFzynwLLLlOXBSTgYyFOPcDpoofyc165VnBbB+nNEH2cbovo7p4+MGMYXiT0mfD8acVZ2o=',
   'x-amz-request-id': 'JR1N0QAM82W3ETVW',
   'date': 'Thu, 10 Mar 2022 20:58:09 GMT',
   'last-modified': 'Thu, 10 Mar 2022 19:17:44 GMT',
   'etag': '"760dcb2a44c5b0553ee12ea8cca057b8"',
   'x-amz-server-side-encryption': 'AES256',
   'accept-ranges': 'bytes',
   'content-type': 'binary/octet-stream',
   'server': 'AmazonS3',
   'content-length': '858'},
  'RetryAttempts': 0},
 'AcceptRanges': 'bytes',
 'LastModified': datetime.datetime(2022, 3, 10, 19, 17, 44, tzinfo=tzutc()),
 'ContentLength': 858,
 'ETag': '"760dcb2a44c5b0553ee12ea8cca057b8"',
 'ContentType': 'binary/octet-stream',
 'ServerSideEncryption': 'AES256',
 'Metadata': {},
 'Body': &lt;botocore.response.StreamingBody at 0x10ad09128&gt;}
</code></pre>
</div>
<p>The actual data for the object is contained in the "Body" of the response. Let's extract that key and read out the data:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="n">txt_body</span> <span class="o">=</span> <span class="n">txt_resp</span><span class="p">[</span><span class="s">"Body"</span><span class="p">].</span><span class="n">read</span><span class="p">().</span><span class="n">decode</span><span class="p">()</span>
<span class="k">print</span><span class="p">(</span><span class="n">txt_body</span><span class="p">)</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight plaintext"><code>The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
</code></pre>
</div>
<h4>More-Complex Files: <code>BytesIO</code></h4>
<p>If we have a file type where we want to do something other than just print out its contents (e.g. images, pickled models, CSVs), we need to add one more step: creating a virtual file with <code>BytesIO</code> (a class in the built-in <a href="https://docs.python.org/3/library/io.html">Python <code>io</code> module</a>). Then we can send that virtual file to a library like Pillow, <code>joblib</code>, or <code>pandas</code>.</p>
<h4>Image File</h4>
<p><code>learn-env</code> already has Pillow, but un-comment the following line if you need to install it in the environment you're currently using to look at this example. Or you can feel free to skip this example and go down to the next one!</p>
<div class="highlight">
<pre class="highlight python"><code><span class="c1"># !conda install pillow -y
</span></code></pre>
</div>
<div class="highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="nn">io</span> <span class="kn">import</span> <span class="n">BytesIO</span>
<span class="kn">from</span> <span class="nn">PIL</span> <span class="kn">import</span> <span class="n">Image</span>

<span class="c1"># Getting the same boto3 image that appears earlier in this lesson
</span><span class="n">img_obj</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="n">Object</span><span class="p">(</span><span class="s">"curriculum-content"</span><span class="p">,</span> <span class="s">"data-science/images/boto3.png"</span><span class="p">)</span>
<span class="c1"># Get the response
</span><span class="n">img_resp</span> <span class="o">=</span> <span class="n">img_obj</span><span class="p">.</span><span class="n">get</span><span class="p">()</span>
<span class="c1"># Read the data into a BytesIO object
</span><span class="n">img_bytes</span> <span class="o">=</span> <span class="n">BytesIO</span><span class="p">(</span><span class="n">img_resp</span><span class="p">[</span><span class="s">"Body"</span><span class="p">].</span><span class="n">read</span><span class="p">())</span>
<span class="c1"># Display the image using Pillow
</span><span class="n">image</span> <span class="o">=</span> <span class="n">Image</span><span class="p">.</span><span class="nb">open</span><span class="p">(</span><span class="n">img_bytes</span><span class="p">)</span>
<span class="n">display</span><span class="p">(</span><span class="n">image</span><span class="p">)</span>
</code></pre>
</div>
<p><img src="https://raw.githubusercontent.com/learn-co-curriculum/dsc-aws-s3/master/index_files/index_18_0.png" alt="png"></p>
<p>This is a bit silly example (displaying image data in a Jupyter notebook) but the same approach could be used for loading data into an image classification tool! Check out <a href="https://medium.com/analytics-vidhya/custom-keras-generator-fetching-images-from-s3-to-train-neural-network-4e98694de8ee">this blog post</a> for an example.</p>
<h4>Pickled Model</h4>
<p>Some model algorithms (e.g. Random Forest, Neural Networks) produce very large pickled models that don't easily fit into GitHub. And in some cases, your deployment approach will not have a file system available, so you'll need to use a cloud storage technique regardless of the model size. Below we show an example of a fairly simple linear regression model that has been stored in S3, but the same concept can easily apply to larger, more-complex models:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="nn">warnings</span>
<span class="n">warnings</span><span class="p">.</span><span class="n">filterwarnings</span><span class="p">(</span><span class="s">'ignore'</span><span class="p">)</span> 

<span class="kn">from</span> <span class="nn">io</span> <span class="kn">import</span> <span class="n">BytesIO</span>
<span class="kn">from</span> <span class="nn">sklearn.linear_model</span> <span class="kn">import</span> <span class="n">LinearRegression</span>
<span class="kn">import</span> <span class="nn">joblib</span>

<span class="c1"># Initialize S3 object
</span><span class="n">pkl_obj</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="n">Object</span><span class="p">(</span><span class="s">"curriculum-content"</span><span class="p">,</span> <span class="s">"data-science/models/regression_model.pkl"</span><span class="p">)</span>
<span class="c1"># Get the response
</span><span class="n">pkl_resp</span> <span class="o">=</span> <span class="n">pkl_obj</span><span class="p">.</span><span class="n">get</span><span class="p">()</span>
<span class="c1"># Read the data into BytesIO object
</span><span class="n">pkl_bytes</span> <span class="o">=</span> <span class="n">BytesIO</span><span class="p">(</span><span class="n">pkl_resp</span><span class="p">[</span><span class="s">"Body"</span><span class="p">].</span><span class="n">read</span><span class="p">())</span>
<span class="c1"># Load the data using joblib
</span><span class="n">loaded_model</span> <span class="o">=</span> <span class="n">joblib</span><span class="p">.</span><span class="n">load</span><span class="p">(</span><span class="n">pkl_bytes</span><span class="p">)</span>
<span class="n">loaded_model</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight plaintext"><code>LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None, normalize=False)
</code></pre>
</div>
<div class="highlight">
<pre class="highlight python"><code><span class="k">print</span><span class="p">(</span><span class="sa">f</span><span class="s">"Loaded model is y = </span><span class="si">{</span><span class="n">loaded_model</span><span class="p">.</span><span class="n">coef_</span><span class="p">[</span><span class="mi">0</span><span class="p">]</span><span class="si">}</span><span class="s">x + </span><span class="si">{</span><span class="n">loaded_model</span><span class="p">.</span><span class="n">intercept_</span><span class="si">}</span><span class="s">"</span><span class="p">)</span>

<span class="n">loaded_model</span><span class="p">.</span><span class="n">predict</span><span class="p">([[</span><span class="mi">10</span><span class="p">],</span> <span class="p">[</span><span class="mi">11</span><span class="p">],</span> <span class="p">[</span><span class="mi">12</span><span class="p">]])</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight plaintext"><code>Loaded model is y = 1.0x + 1.0





array([11., 12., 13.])
</code></pre>
</div>
<h4>CSV File</h4>
<p>CSV data is a more realistic file type you might be working with. The below example reads a CSV file from an S3 bucket, then loads it into <code>pandas</code>:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="kn">from</span> <span class="nn">io</span> <span class="kn">import</span> <span class="n">BytesIO</span>
<span class="kn">import</span> <span class="nn">pandas</span> <span class="k">as</span> <span class="n">pd</span>

<span class="c1"># Initialize S3 object
</span><span class="n">csv_obj</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="n">Object</span><span class="p">(</span><span class="s">"curriculum-content"</span><span class="p">,</span> <span class="s">"data-science/data/test.csv"</span><span class="p">)</span>
<span class="c1"># Get the response
</span><span class="n">csv_resp</span> <span class="o">=</span> <span class="n">csv_obj</span><span class="p">.</span><span class="n">get</span><span class="p">()</span>
<span class="c1"># Read the data into a BytesIO object
</span><span class="n">csv_bytes</span> <span class="o">=</span> <span class="n">BytesIO</span><span class="p">(</span><span class="n">csv_resp</span><span class="p">[</span><span class="s">"Body"</span><span class="p">].</span><span class="n">read</span><span class="p">())</span>
<span class="c1"># Load the data with pandas
</span><span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="n">read_csv</span><span class="p">(</span><span class="n">csv_bytes</span><span class="p">)</span>
<span class="n">df</span>
</code></pre>
</div>
<div>
<table class="dataframe" border="1">
<thead>
<tr style="text-align: right;">
<th></th>
<th>id</th>
<th>battery_power</th>
<th>blue</th>
<th>clock_speed</th>
<th>dual_sim</th>
<th>fc</th>
<th>four_g</th>
<th>int_memory</th>
<th>m_dep</th>
<th>mobile_wt</th>
<th>...</th>
<th>pc</th>
<th>px_height</th>
<th>px_width</th>
<th>ram</th>
<th>sc_h</th>
<th>sc_w</th>
<th>talk_time</th>
<th>three_g</th>
<th>touch_screen</th>
<th>wifi</th>
</tr>
</thead>
<tbody>
<tr>
<th>0</th>
<td>1</td>
<td>1043</td>
<td>1</td>
<td>1.8</td>
<td>1</td>
<td>14</td>
<td>0</td>
<td>5</td>
<td>0.1</td>
<td>193</td>
<td>...</td>
<td>16</td>
<td>226</td>
<td>1412</td>
<td>3476</td>
<td>12</td>
<td>7</td>
<td>2</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>1</th>
<td>2</td>
<td>841</td>
<td>1</td>
<td>0.5</td>
<td>1</td>
<td>4</td>
<td>1</td>
<td>61</td>
<td>0.8</td>
<td>191</td>
<td>...</td>
<td>12</td>
<td>746</td>
<td>857</td>
<td>3895</td>
<td>6</td>
<td>0</td>
<td>7</td>
<td>1</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>2</th>
<td>3</td>
<td>1807</td>
<td>1</td>
<td>2.8</td>
<td>0</td>
<td>1</td>
<td>0</td>
<td>27</td>
<td>0.9</td>
<td>186</td>
<td>...</td>
<td>4</td>
<td>1270</td>
<td>1366</td>
<td>2396</td>
<td>17</td>
<td>10</td>
<td>10</td>
<td>0</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<th>3</th>
<td>4</td>
<td>1546</td>
<td>0</td>
<td>0.5</td>
<td>1</td>
<td>18</td>
<td>1</td>
<td>25</td>
<td>0.5</td>
<td>96</td>
<td>...</td>
<td>20</td>
<td>295</td>
<td>1752</td>
<td>3893</td>
<td>10</td>
<td>0</td>
<td>7</td>
<td>1</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>4</th>
<td>5</td>
<td>1434</td>
<td>0</td>
<td>1.4</td>
<td>0</td>
<td>11</td>
<td>1</td>
<td>49</td>
<td>0.5</td>
<td>108</td>
<td>...</td>
<td>18</td>
<td>749</td>
<td>810</td>
<td>1773</td>
<td>15</td>
<td>8</td>
<td>7</td>
<td>1</td>
<td>0</td>
<td>1</td>
</tr>
<tr>
<th>...</th>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<th>995</th>
<td>996</td>
<td>1700</td>
<td>1</td>
<td>1.9</td>
<td>0</td>
<td>0</td>
<td>1</td>
<td>54</td>
<td>0.5</td>
<td>170</td>
<td>...</td>
<td>17</td>
<td>644</td>
<td>913</td>
<td>2121</td>
<td>14</td>
<td>8</td>
<td>15</td>
<td>1</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>996</th>
<td>997</td>
<td>609</td>
<td>0</td>
<td>1.8</td>
<td>1</td>
<td>0</td>
<td>0</td>
<td>13</td>
<td>0.9</td>
<td>186</td>
<td>...</td>
<td>2</td>
<td>1152</td>
<td>1632</td>
<td>1933</td>
<td>8</td>
<td>1</td>
<td>19</td>
<td>0</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<th>997</th>
<td>998</td>
<td>1185</td>
<td>0</td>
<td>1.4</td>
<td>0</td>
<td>1</td>
<td>1</td>
<td>8</td>
<td>0.5</td>
<td>80</td>
<td>...</td>
<td>12</td>
<td>477</td>
<td>825</td>
<td>1223</td>
<td>5</td>
<td>0</td>
<td>14</td>
<td>1</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>998</th>
<td>999</td>
<td>1533</td>
<td>1</td>
<td>0.5</td>
<td>1</td>
<td>0</td>
<td>0</td>
<td>50</td>
<td>0.4</td>
<td>171</td>
<td>...</td>
<td>12</td>
<td>38</td>
<td>832</td>
<td>2509</td>
<td>15</td>
<td>11</td>
<td>6</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>999</th>
<td>1000</td>
<td>1270</td>
<td>1</td>
<td>0.5</td>
<td>0</td>
<td>4</td>
<td>1</td>
<td>35</td>
<td>0.1</td>
<td>140</td>
<td>...</td>
<td>19</td>
<td>457</td>
<td>608</td>
<td>2828</td>
<td>9</td>
<td>2</td>
<td>3</td>
<td>1</td>
<td>0</td>
<td>1</td>
</tr>
</tbody>
</table>
<p>1000 rows × 21 columns</p>
</div>
<h4>Avoiding <code>BytesIO</code> by Installing S3Fs</h4>
<p>If you are using <code>pandas</code> and you want to avoid having to use <code>BytesIO</code>, there is a library called <a href="https://s3fs.readthedocs.io/en/latest/">S3Fs (S3 Filesystem)</a> that can help you shorten the above code by treating S3 URLs as regular file system URLs. You just need to start the file path with <code>"s3://"</code>!</p>
<p>(S3Fs is used "under the hood" by <code>pandas</code> and does not require a separate import for this type of use.)</p>
<div class="highlight">
<pre class="highlight python"><code><span class="c1"># !conda install s3fs -c conda-forge -y
</span></code></pre>
</div>
<p>Here is the same example as above, using S3Fs to shorten the code:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="kn">import</span> <span class="nn">pandas</span> <span class="k">as</span> <span class="n">pd</span>
<span class="n">df</span> <span class="o">=</span> <span class="n">pd</span><span class="p">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s">"s3://curriculum-content/data-science/data/test.csv"</span><span class="p">)</span>
<span class="n">df</span>
</code></pre>
</div>
<div>
<table class="dataframe" border="1">
<thead>
<tr style="text-align: right;">
<th></th>
<th>id</th>
<th>battery_power</th>
<th>blue</th>
<th>clock_speed</th>
<th>dual_sim</th>
<th>fc</th>
<th>four_g</th>
<th>int_memory</th>
<th>m_dep</th>
<th>mobile_wt</th>
<th>...</th>
<th>pc</th>
<th>px_height</th>
<th>px_width</th>
<th>ram</th>
<th>sc_h</th>
<th>sc_w</th>
<th>talk_time</th>
<th>three_g</th>
<th>touch_screen</th>
<th>wifi</th>
</tr>
</thead>
<tbody>
<tr>
<th>0</th>
<td>1</td>
<td>1043</td>
<td>1</td>
<td>1.8</td>
<td>1</td>
<td>14</td>
<td>0</td>
<td>5</td>
<td>0.1</td>
<td>193</td>
<td>...</td>
<td>16</td>
<td>226</td>
<td>1412</td>
<td>3476</td>
<td>12</td>
<td>7</td>
<td>2</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>1</th>
<td>2</td>
<td>841</td>
<td>1</td>
<td>0.5</td>
<td>1</td>
<td>4</td>
<td>1</td>
<td>61</td>
<td>0.8</td>
<td>191</td>
<td>...</td>
<td>12</td>
<td>746</td>
<td>857</td>
<td>3895</td>
<td>6</td>
<td>0</td>
<td>7</td>
<td>1</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>2</th>
<td>3</td>
<td>1807</td>
<td>1</td>
<td>2.8</td>
<td>0</td>
<td>1</td>
<td>0</td>
<td>27</td>
<td>0.9</td>
<td>186</td>
<td>...</td>
<td>4</td>
<td>1270</td>
<td>1366</td>
<td>2396</td>
<td>17</td>
<td>10</td>
<td>10</td>
<td>0</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<th>3</th>
<td>4</td>
<td>1546</td>
<td>0</td>
<td>0.5</td>
<td>1</td>
<td>18</td>
<td>1</td>
<td>25</td>
<td>0.5</td>
<td>96</td>
<td>...</td>
<td>20</td>
<td>295</td>
<td>1752</td>
<td>3893</td>
<td>10</td>
<td>0</td>
<td>7</td>
<td>1</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>4</th>
<td>5</td>
<td>1434</td>
<td>0</td>
<td>1.4</td>
<td>0</td>
<td>11</td>
<td>1</td>
<td>49</td>
<td>0.5</td>
<td>108</td>
<td>...</td>
<td>18</td>
<td>749</td>
<td>810</td>
<td>1773</td>
<td>15</td>
<td>8</td>
<td>7</td>
<td>1</td>
<td>0</td>
<td>1</td>
</tr>
<tr>
<th>...</th>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<th>995</th>
<td>996</td>
<td>1700</td>
<td>1</td>
<td>1.9</td>
<td>0</td>
<td>0</td>
<td>1</td>
<td>54</td>
<td>0.5</td>
<td>170</td>
<td>...</td>
<td>17</td>
<td>644</td>
<td>913</td>
<td>2121</td>
<td>14</td>
<td>8</td>
<td>15</td>
<td>1</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>996</th>
<td>997</td>
<td>609</td>
<td>0</td>
<td>1.8</td>
<td>1</td>
<td>0</td>
<td>0</td>
<td>13</td>
<td>0.9</td>
<td>186</td>
<td>...</td>
<td>2</td>
<td>1152</td>
<td>1632</td>
<td>1933</td>
<td>8</td>
<td>1</td>
<td>19</td>
<td>0</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<th>997</th>
<td>998</td>
<td>1185</td>
<td>0</td>
<td>1.4</td>
<td>0</td>
<td>1</td>
<td>1</td>
<td>8</td>
<td>0.5</td>
<td>80</td>
<td>...</td>
<td>12</td>
<td>477</td>
<td>825</td>
<td>1223</td>
<td>5</td>
<td>0</td>
<td>14</td>
<td>1</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>998</th>
<td>999</td>
<td>1533</td>
<td>1</td>
<td>0.5</td>
<td>1</td>
<td>0</td>
<td>0</td>
<td>50</td>
<td>0.4</td>
<td>171</td>
<td>...</td>
<td>12</td>
<td>38</td>
<td>832</td>
<td>2509</td>
<td>15</td>
<td>11</td>
<td>6</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>999</th>
<td>1000</td>
<td>1270</td>
<td>1</td>
<td>0.5</td>
<td>0</td>
<td>4</td>
<td>1</td>
<td>35</td>
<td>0.1</td>
<td>140</td>
<td>...</td>
<td>19</td>
<td>457</td>
<td>608</td>
<td>2828</td>
<td>9</td>
<td>2</td>
<td>3</td>
<td>1</td>
<td>0</td>
<td>1</td>
</tr>
</tbody>
</table>
<p>1000 rows × 21 columns</p>
</div>
<h3>Connecting to Your S3 Bucket</h3>
<p>In the cell below, replace the string values with the names of your S3 bucket and the file you want to read:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="n">bucket_name</span> <span class="o">=</span> <span class="s">""</span>
<span class="n">object_name</span> <span class="o">=</span> <span class="s">""</span>
</code></pre>
</div>
<p>Now we'll attempt to load your object using <code>boto3</code>:</p>
<div class="highlight">
<pre class="highlight python"><code><span class="n">obj</span> <span class="o">=</span> <span class="n">s3</span><span class="p">.</span><span class="n">Object</span><span class="p">(</span><span class="n">bucket_name</span><span class="p">,</span> <span class="n">object_name</span><span class="p">)</span>
<span class="n">obj</span>
</code></pre>
</div>
<div class="highlight">
<pre class="highlight python"><code><span class="n">response</span> <span class="o">=</span> <span class="n">obj</span><span class="p">.</span><span class="n">get</span><span class="p">()</span>
<span class="n">response</span>
</code></pre>
</div>
<p>(If you get a <code>NoCredentialError</code> here, that means that you either have the wrong bucket or object name, or you did not set the permissions on the bucket or object correctly. Double-check that the steps above are working as expected.)</p>
<p>Now you can continue with whatever next steps are appropriate for your use case!</p>
<h2>Summary</h2>
<p>In this lesson, we introduced the use cases for S3 in data science: typically data scientists use S3 for storing files that are too big for GitHub, but still need to be accessible from Python code for the purposes of reproducibility, using certain cloud services, or deployment. These can include many types of files, including text, images, pickled models, and data files such as CSV.</p>
<p>We recommend that you use the AWS console to upload content to a bucket and configure the access permissions, then use the <code>boto3</code> Python library to access this content from the context of Python code.</p>