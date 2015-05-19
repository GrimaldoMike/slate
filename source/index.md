---
title: API Reference

language_tabs:
  - ruby + rails

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Instahash API! You can use our API to access all of you Facebook photos and see them in one convinient place.

We integrated Facebook's Graph API with the convinient  Fb_Graph2 gem, that allow developers to manage data simply and well organized.

You can check Fb_Graph2 API at [Fb_Graph2](https://github.com/nov/fb_graph2) 

This example API documentation page was created with [Slate](http://github.com/tripit/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Controllers

The Instahash API is all about controlling the information of the user and manage it in a simple-restful way.

## AlbumsController
> 	def index
<br>&nbsp;		fb_login = auth
<br>&nbsp;		if fb_login[:status_code] == 200
<br>&nbsp;&nbsp;			response = fb_login[:data].albums
<br>&nbsp;		else
<br>&nbsp;&nbsp;			response = fb_login[:data]
<br>&nbsp;		end	
<br>&nbsp;		respond_to do |format|
<br>&nbsp;&nbsp;			format.json { render :json => response, status: fb_login[:status_code]}
<br>&nbsp;		end
<br>	end
	
*<aside>
The AlbumsController authenticates the Facebook user and stores data
</aside>*

>		def auth
<br>&nbsp;			begin
<br>&nbsp;&nbsp;				user = FbGraph2::User.new(params[:user_id]).authenticate(params[:access_token])
<br>&nbsp;&nbsp;				user.fetch
<br>&nbsp;&nbsp;				status_code = 200
<br>&nbsp;&nbsp;				return {:data => user, :status_code => status_code}
<br>&nbsp;			rescue => ex
<br>&nbsp;&nbsp;			   ex.message
<br>&nbsp;&nbsp;			   status_code = 400
<br>&nbsp;&nbsp;			   return {:data => ex.message, :status_code => status_code}
<br>&nbsp;			end
<br>		end


<p>The FbGraph2 Api gets the main fields from the Facebook Api User request and displays his Albums </p>

`user = FbGraph2::User.new(params[:user_id]).authenticate(params[:access_token])`
<br>
*<aside>
The albums_controller_spec verifies that the Controller responds accordingly
</aside>*
<p>The album controller spec checks that the controller is:</p>
<ul>
  <li>Properly versioned </li>
  <li> Responds to an user_id and access_token </li>
  <li> It's not an unauthorized access </li>
  <li> Returns a JSON </li>
</ul>


## PhotosController

>	def indexcd ....
<br>&nbsp;		response = photos
<br>&nbsp;		respond_to do |format|
<br>&nbsp;&nbsp;			format.json { render :json => response[:data], status: response[:status_code]}
<br>&nbsp;		end
<br>&nbsp;	end

>		def photos
<br>&nbsp;			begin
<br>&nbsp;&nbsp;				album = FbGraph2::Album.new(params[:album_id]).authenticate(params[:access_token]).fetch.photos
<br>&nbsp;&nbsp;				photos = []
<br>&nbsp;&nbsp;				album.each do |photo|
<br>&nbsp;&nbsp;&nbsp;					if photo.name
<br>&nbsp;&nbsp;&nbsp;&nbsp;						matches = (photo.name).scan(/\B#\w*[\u00F1A-Za-z_]+\w*/)
<br>&nbsp;&nbsp;&nbsp;					else
<br>&nbsp;&nbsp;&nbsp;&nbsp;						matches = []
<br>&nbsp;&nbsp;&nbsp;					end
<br>&nbsp;&nbsp;&nbsp;					tag = Api::V1::Tag.where(photo_id:photo.id)
<br>&nbsp;&nbsp;&nbsp;					local_tags = []
<br>&nbsp;&nbsp;&nbsp;					tag.each do |t|
<br>&nbsp;&nbsp;&nbsp;&nbsp;						local_tags.push(t.tag)
<br>&nbsp;&nbsp;&nbsp;					end
<br>&nbsp;&nbsp;&nbsp;				   photos.push({:id => photo.id, :picture => photo.picture, :name => photo.name, :link => photo.link, :tags => matches, :local_tags => local_tags})
<br>&nbsp;&nbsp;&nbsp;				end
<br>&nbsp;&nbsp;				status_code = 200
<br>&nbsp;&nbsp;				return {:data => photos, :status_code => status_code}
<br>&nbsp;			rescue => ex
<br>&nbsp;&nbsp;			   ex.message
<br>&nbsp;&nbsp;			   status_code = 400
<br>&nbsp;	&nbsp;		   return {:data => ex.message, :status_code => status_code}
<br>&nbsp;			end
<br>		end

*<aside>
The PhotosController obtains a collection of Photos giving a certain Album.
</aside>*
<p> Again, The FbGraph2 Api gets the fields from the Facebook API, but this time from not from the User. 
    Instead, it uses the Album to get the photos of a certain album and gets its attributes (name, link, picture).  </p>


		
`album = FbGraph2::Album.new(params[:album_id]).authenticate(params[:access_token]).fetch.photos`
<br>
*<aside>
The photos_controller_spec verifies that the Controller responds accordingly
</aside>*
<p>The photo controller spec checks that the controller is:</p>
<ul>
  <li>Properly versioned </li>
  <li> Responds to an album_id and access_token </li>
  <li> It's not an unauthorized access </li>
  <li> Returns a JSON </li>
</ul>


## TagsController
>  def create
<br>&nbsp;  	if params[:tag][:photo_id]
<br>&nbsp;&nbsp;  		tag = Api::V1::Tag.new(tag_params)
<br>&nbsp;&nbsp;  		respond_to do |format|
<br>&nbsp;&nbsp;&nbsp;  			if tag.save
<br>&nbsp;&nbsp;&nbsp;&nbsp;  				format.json {render :json => tag, status: 201}
<br>&nbsp;&nbsp;&nbsp;  			else
<br>&nbsp;&nbsp;  				format.jsont {render :json => "An error ocurred"}
<br>&nbsp;&nbsp;  			end
<br>&nbsp;  		end
<br>  	end

>  def destroy
<br>&nbsp;  	if params[:tag][:photo_id]
<br>&nbsp;&nbsp;  		tag = Api::V1::Tag.where(photo_id:params[:tag][:photo_id], tag:params[:tag][:tag]).first
<br>&nbsp;&nbsp;  		tag.destroy
<br>&nbsp;&nbsp;  		respond_to do |format|
<br>&nbsp;&nbsp;&nbsp;			format.json { head 204 }
<br>&nbsp;&nbsp;		  end
<br>&nbsp;  	  end
<br>    end
  
*<aside>
The TagsController Inserts and Deletes Tag records.
</aside>*
<p> This Controller Creates new instances of the Tag class and saves them in an Active Record. 
    Also is in charge of Destroying instances of Tag object obtained from the database. 
    After doing either action, the Tag Controller returns the object that was modified.</p>

`CREATE  :: tag = Api::V1::Tag.new(tag_params)`
<br>
<br>
`DESTROY :: tag = Api::V1::Tag.where(photo_id:params[:tag][:photo_id], tag:params[:tag][:tag]).first`
<br>
*<aside>
The photos_controller_spec verifies that the Controller responds accordingly
</aside>*
<p>The photo controller spec checks that the controller is:</p>
<ul>
  <li>Properly versioned </li>
  <li> Responds with a Tag object in JSON </li>
  <li> It's not an unauthorized access </li>
  <li> Returns a JSON </li>
</ul>

