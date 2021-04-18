+++
aliases = []
date = 2021-04-18T12:59:00Z
lastmod = 2021-04-18T12:59:00Z
publishdate = 2021-04-18T12:59:00Z
tags = ["java", "temporary", "files"]
title = "Temporary Files 2"

+++
This is Part 2. Please read Part 1 [here](https://anirudhtest.netlify.app/temporary-files-1/).

## Use Case

Suppose your manager tells you - "Yo Sadio, write me a tool that takes a URL and uploads its content to s3."

## Thinking Process

Well, he doesn't care where you download it, how you upload it, or what you do with the content later on.

## The how?

The tempfile is deleted manually in [temporary-files-1](https://anirudhtest.netlify.app/temporary-files-1/).

This version uses try-with-resources to delete it.

1. Create a static class (TempFile) that implements the "Closeable" interface. 
2. Create the tempfile object with try and resources. 
3. Now you have the handle, perform an action, and close the block. 
4. That's it, you don't have to worry about deleting the file since close() will be called right after the try block is done.

## Code

    // Try with Resources
    public static void main(String[] args) throws IOException {
        // 0. Setup!
        File file = File.createTempFile("prefix", null);
    
        String urlString = "https://gist.github.com/myusuf3/7f645819ded92bda6677";
        URL url = new URL(urlString);
    
        try (TempFile tempFile = new TempFile(file)) {
            // 1. download to local
            FileUtils.copyURLToFile(url, tempFile.getFile());
    
            // 2. upload to s3
            PutObjectRequest por = new PutObjectRequest(BUCKET_NAME, file.getName(), file);
            amazonS3Client.putObject(por);
        }
    }
    
    public static class TempFile implements Closeable {
        private final File file;
        public TempFile(File file) {
            this.file = file;
        }
        public File getFile() {
            return file;
        }
        @Override
        public void close() {
            if (file.delete()) {
                System.out.println("Deleted the file: " + file.getAbsolutePath());
            } else {
                System.out.println("Could not delete the file: " + file.getAbsolutePath());
            }
        }
    }

That's it, simple.