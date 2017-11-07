---
layout: post
title: "File uploads via GraphQL mutations with a Rails server"
date: 2017-11-05 18:25:10 -0700
categories: ruby rails code
---

# rails_apollo_fetch_upload_middleware

## The Problem

I was trying to set up file uploading in my GraphQL+Ruby/Rails+Redux/Rect project. I wanted to use GraphQL mutations to upload
files to the server. I prefered if I could use multipart forms instead of having to base64-encode the file contents, as the
latter would take up 4/3 the space and increase my CPU usage. Besides, forms make uploading more natural and I was
wondering what it would look like to support basic form-enabled file uploading in a GraphQL world with a Rails backend. I soon
found out that no one (as far as I can tell) has done this.

## The Solution

The closest I could find was a very neat npm package called 
[apollo-fetch-upload](https://github.com/apollographql/apollo-fetch/tree/master/packages/apollo-fetch-upload). The package
author made it so that files which are uploaded client-side can be sent using the Apollo client (once you setup the
`apollo-fetch-upload` stuff when you create your `ApolloClient` instance). On the client, you can simply set the native File
instance as a GraphQL input argument:

```js

const FileUploadWrapper = ({ mutate }) => (
  <FileUploader onAttachedFile={(file, title) => mutate({ variables: { file, title } })} />
);

export default graphql(gql`
  mutation($file: Upload!, $title: String!) {
    uploadDoc(file: $file, title: $title) {
      id
      title
      doc {
        url
      }
    }
  }
`)(FileUploadWrapper);
```

Where the `Upload` GraphQL type is (basically) this (my best guess at a flow type for it):

```
name: String!
size: Int!
path: String!
type: String!
```

However, to use this package on the client, it assumes you are (1) using NodeJS on the server/backend and (2) will then use
the accompanying [apollo-upload-server](https://github.com/jaydenseric/apollo-upload-server) middleware. This middleware 
basically changes the incoming request to populate the `file` input variables (of type `Upload`) on the mutation with the
`name` which comes from the form and the `path`, `size`, and `type` which come from the temp files on the server which are
created as a result of the form post.

## Rails is still cool!

I'm using Rails on the backend because it's what I know well and I want to move as fast as possible on my project. I just
frankly don't have the patience to learn how to do everything in express or some other NodeJS web server (I haven't even
thought about ORMs!).

I'm also using the [graphql gem](https://github.com/rmosolgo/graphql-ruby), which is a very nice GraphQL server implementation
for Ruby. It has worked very well so far us for all of our queries and other non-file-uploading mutations along with Apollo
so we'd like to keep using it. Thus, I decided to
[create a Rails middleware gem](https://github.com/abeland/apollo_fetch_upload_rails_middleware) which populates the
GraphQL mutation input variables upon file uploading similar to how `apollo-upload-server` does it. It does __not__ handle
batching yet, but I may add that later when/if I need it.

I have tested it on single uploads of files and we use it to create AR models which have Paperclip attachments storing the
file contents in S3. So far, so good. If you are here because you're looking to handle file upload GraphQL mutations in Rails,
I hope you'll try my gem out and file an issues you might find or even submit a PR to improve it. Or, just enjoy the middleware!
