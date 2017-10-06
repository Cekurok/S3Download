# S3Download

try {
            TransferManager tm =  TransferManagerBuilder.standard()
                    .withS3Client(s3Client)
                    .build();  // создание клиента S3 с нужнеми параметрами



            ListObjectsRequest listObjectsRequest = new ListObjectsRequest().withBucketName(bucketName); // запрос списка файлов в бакете
            ObjectListing objectListing = s3Client.listObjects(listObjectsRequest);
            ArrayList<String> lines = new ArrayList<String>();

            for (S3ObjectSummary s3ObjectSummary : objectListing.getObjectSummaries()) {
                String file = s3ObjectSummary.getKey();
                File ff = new File("E:\\\\TestDownload\\" + file);
                FileOutputStream outFile = new FileOutputStream(ff,true);
                S3Object objectDownload = s3Client.getObject(new GetObjectRequest(bucketName,file));
                InputStream objData = objectDownload.getObjectContent();
                File newBufFile = new File("E:\\\\TestDownload\\buffer");
                FileWriter sw = new FileWriter(ff,true);
                if (ff.exists() && (ff.length() < s3ObjectSummary.getSize())){
                   try {
                       BufferedReader reader = new BufferedReader(new InputStreamReader(objData));
                       String line = reader.readLine();
                       while((line) != null){
                           sw.write(line);
                       }
                   } catch (IOException ex){
                       result.sampleEnd(); // stop stopwatch
                       result.setSuccessful(false);
                       result.setResponseMessage("Exception: " + ex);

                       // get stack trace as a String to return as document data
                       java.io.StringWriter stringWriter = new java.io.StringWriter();
                       ex.printStackTrace(new java.io.PrintWriter(stringWriter));
                       result.setDataType("text");
                       result.setResponseCode("500");
                   }




//                    OutputStream outStream = new FileOutputStream(ff, true);  // попытка загрузить по байтно очень медленно
//                    byte[] buffer = new byte[10048576];
//                    ByteArrayOutputStream byteWriter = new ByteArrayOutputStream();
//                    while (true){
//                        int readBytesCount = objData.read(buffer);
//                        if (readBytesCount == -1){
//                            break;
//                        }
//                        if (readBytesCount > 0){
//                            outStream.write(readBytesCount);
//                        }
//                    }
//                    outStream.flush();
//                    outStream.close();

//                    OutputStream outStream = new FileOutputStream(ff, true);
//                    outStream.write(objData.read());


                  

                } else {
                    if (s3ObjectSummary.getSize() > 0) {
                        FileUtils.copyInputStreamToFile(objData,ff);

                }
                }
//                   
            }
              
            tm.shutdownNow();
            s3Client.shutdown();



/

            result.sampleEnd();
            result.setSuccessful(true);


            result.setResponseCodeOK();
        } catch (AmazonServiceException ase) {
            System.err.println(ase.getMessage());
            ase.printStackTrace(System.err);
            System.out.println("Caught an AmazonServiceException, which " +
                    "means your request made it " +
                    "to Amazon S3, but was rejected with an error response" +
                    " for some reason.");
            System.out.println("Error Message:    " + ase.getMessage());
            System.out.println("HTTP Status Code: " + ase.getStatusCode());
            System.out.println("AWS Error Code:   " + ase.getErrorCode());
            System.out.println("Error Type:       " + ase.getErrorType());
            System.out.println("Request ID:       " + ase.getRequestId());
            System.exit(1);

        } catch (AmazonClientException err) {
            System.err.println(err.getMessage());
            err.printStackTrace(System.err);
            System.out.println("Caught an AmazonClientException, which " +
                    "means the client encountered " +
                    "an internal error while trying to " +
                    "communicate with S3, " +
                    "such as not being able to access the network.");
            System.out.println("Error Message: " + err.getMessage());
            System.exit(1);

        } catch (Exception e) {
            result.sampleEnd(); // stop stopwatch
            result.setSuccessful(false);
            result.setResponseMessage("Exception: " + e);

            // get stack trace as a String to return as document data
            java.io.StringWriter stringWriter = new java.io.StringWriter();
            e.printStackTrace(new java.io.PrintWriter(stringWriter));
            result.setDataType("text");
            result.setResponseCode("500");
        }

        return result;
    }
