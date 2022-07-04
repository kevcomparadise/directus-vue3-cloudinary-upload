# Cloudinary Directus Extension (Vue3)
Simple interface for upload/read image on your cloudinary

## Installation
1. Download or fork the repository
2. Install the requirements
   ```npm install``` or ```yarn```
3. Copy .env.example to .env and replace variables with your cloudname, apikey and apisecret
4. Build the extension ```npm run build```  or ```yarn build``` 
5. Create a folder in your directus extension/interface folder named **cloudinary-upload** or an alternate route name.
6. Move the index.js build file to your new folder **directus/extensions/interface/cloudinary-upload/index.js**
7. Restart directus !

## How to use
1. Create new standard field with `string` type
2. For interface select **Cloudinary Upload**.

## TODO
- [ ] Update field value (get JSON instead of link)
- [ ] Interface option
