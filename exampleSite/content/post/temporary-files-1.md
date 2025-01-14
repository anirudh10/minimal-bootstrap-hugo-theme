+++
aliases = []
date = 2021-04-17T22:59:51Z
lastmod = 2021-04-17T22:59:00Z
publishdate = 2021-04-17T22:59:00Z
tags = ["java", "temporary", "files"]
title = "Temporary Files 1"

+++
## Use Case

Suppose your manager tells you - "Yo Sadio, write me a tool that takes a URL and uploads its content to s3."

## Thinking Process

Well, he doesn't care where you download it, how you upload it, or what you do with the content later on.

## You need to

1. Create a temporary file.
2. Download the URL contents to it.
3. Delete the file.

## Code

    // 0. setup
    File file = File.createTempFile("prefix", null);
    
    String urlString = "https://gist.github.com/myusuf3/7f645819ded92bda6677";
    URL url = new URL(urlString);
    
    // 1. download to local
    FileUtils.copyURLToFile(url, file);
    System.out.println(file.getName());
    
    // 2. upload to s3
    PutObjectRequest por = new PutObjectRequest(BUCKET_NAME, file.getName(), file);
    amazonS3Client.putObject(por);
    
    // 3. delete the file
    System.out.println(file.delete());

That's it, there is temporary-files-2 that goes one step deeper.