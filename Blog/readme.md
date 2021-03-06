# How to create PDFs on the fly with Umbraco.
## Setup

I had the requirement from a client to create a PDF of a page. The page had to be CMSd and the PDF should be dynamic. The site has what we
called a basket. The visitor to the website could select items that they wanted to save for later reference in to the basket. Then the 
PDF could be created on the fly, pulling in the items from the cart and also pulling in the CMS'd header, footer and other items.

In this tutorial, I'll cover the PDF setup and the Umbraco setup. The basket was saved as a JSON object, deserialized and added to the PDF.

# Umbraco Setup.
I originally created this in an Umbraco 7 website but this tutorial will be for Umbraco 8. It will be on a clean install and the code is attached below.

Let's open up Visual Studio and use Nuget to install Umbraco CMS. For detailed instructions on setting up Umbraco, visit [our.umbraco.com](https://our.umbraco.com/download/)

The project that I've included in this tutorial has the following login details:

Admin: admin@pdfproject.com

Password: pdfproject

I have two projects within my Visual Studio Solution. UmbracoPDF and UmbracoPDF.Core. 

`UmbracoPDF.Core` has the following references added, all these files are also within the `UmbracoPDF` project and so I just add a reference from the `UmbracoPDF` bin folder to the .Core project:

`Umbraco.Core`
`Umbraco.Web`
`System.Web.Http`
`System.Web.Mvc`
`Rotativa`

I have also added `Umbraco.ModelsBuilder.Embedded` because I have moved the default Models location from the main project to my `.Core` project so that I can use the models with my controllers. 

# How will we create PDFs?
I used [Rotativa](https://github.com/webgio/Rotativa) 1.7.3, which is a free nuget package which can be used in ASP.net MVC projects. When it's installed, it needs to be in the UmbracoPDF project. There is a reason for this which I'll explain in a bit.  
 
So we have Umbraco installed and Rotativa installed. Now we need to hook it all together. 

Let's start by making an external website in to a PDF. 
This will demonstrate that Rotativa is installed correctly and generating PDFs.

The View:


![Basic View](/Blog/Images/pdfView.png)


The Controller:

 ![Basic Controller](/Blog/Images/pdfController.png)
 
 When you load up the project, all this will do is place a link on the homepage that a user can click. When the user clicks the link Rotativa goes away and reads the website and converts it to a PDF which then downloads on to your computer. For this example, the PDF will download with the name `OwainCodes.PDF`. We now have Rotativa working and it's time to make it work with some Umbraco content.
 
 ## Umbraco Content and Rotativa PDF
 
My plan here is to make what I call a Content Block. It's a Nested Content element which I will drop on to a page. This makes the element optional and I can place it on any page that I want. 
The first thing I need to do is setup my Nested Content element within Umbraco. In the backoffice I've created two folders within my Document Type folder, which can be found under Settings, one folder for my compositions and another for my Content Blocks.This isn't required but it can help with keeping things organised, especially as your site grows in size.  

 ![ContentBlocksCompositions](/Blog/Images/contentBlocksCompositions.jpg)
 
 You need to create your Nested Content Element (Content Block) first, because without it, you can't add anything to the Nested Content property editor. The composition `Page Content` has the Nested Content property editor on it and that's it. The Nested Content Property Editor just now only has one element setup on it, the PDF download button element. 

I then add the composition `Page Content` to my Home Document Type. 

 ![HomeDocumentType](/Blog/Images/HomeDocumentType.jpg)
 
We are almost there. We just need to wire this all up with some code. 
Obviously you can add more elements to this page over time and I'll add some more later in this tutorial to demonstrate the PDF creation but just now, all I want to do is make sure my element is displaying on the homepage correctly.

Here is how my Homepage now looks with everything setup.

 ![HomeDocumentType](/Blog/Images/ContentBlocksDemo.gif)

With the backoffice now setup. I can build out my Views.

I need 3 files setup. 
* `Home` view
* `ContentBlocks` view
* `PDFDownloadButton` view

The `Home`view calls the `ContentBlocks` view and the `ContentBlocks` view call the `PDFDownloadButton` view. 
The reason for this is because the Home view has it's own models and so does the PDFDownloadButton view. I can't convert from one model type to another without going through the ContentBlocks view, which has a generic type set on it.

Here is the code: 

Home.cshtml
![Home View](/Blog/Images/HomeView.png)

As you can see from the code above, I have placed my `DisplayContentBlock.cshtml` in to a new folder called `ContentBlocks`. 

DisplayContentBlocks.cshtml
![DisplayContentBlock](/Blog/Images/DisplayContentBlock.png)

This switch statement will grow over time but by keeping everything in this one file, I can add as many elements as I like to any page on my site. This one just calls the `PDFDownloadButton` which is also saved within the ContentBlocks folder.

PDFDownloadButton.cshtml
![Display Download Button](/Blog/Images/PDFDownloadButton.png)

This reads the text that is entered in the backoffice and displays it as a link on the frontend. Just now, it doesn't have a `href` assigned to it, all I want just now is for the text to display on the frontend. 

![Testing Nested Content](/Blog/Images/basicNestedContentTest.jpg)

Success, we have the link that we created to test the PDF functionality and now we have the link that is created from Nested Content from the backoffice. The final step is to hook the two together. A link that creates a PDF, controlled by the backoffice.

## PDF any page with a custom template

The output of this next section is to have a custom template which renders a PDF of the page, however, it's not going to show the navigation, the footer or anything like that. The PDF will show only the content of the page withinn a specific div but allow for the PDF to be controlled via the backoffice e.g. add an advert, add some custom text, anything you want. 

I've created a new `.cshtml` file which is just for the PDF. It's called `PDFTemplate.cshtml`. 
The issue I had was, how do I pass the page content to a seperate view. For this, I used javascript. I look for a specific div id then store that information as a string of HTML and pass it to my new template view.

Step 1. Get the content of a div and store it in local storage.
Step 2. Place the information in to form, then pass this infomraiton to a controller.

Code screenshots are by https://carbon.now.sh/
