# All CRUD Here
## Resource CURD (-mcr) 
###### Here we create custom image upload
 First of all you need to put this bellow code  **Base Controloler**
 <pre>
  public function uploadImage($prefix,$image,$path){
        $image_name=$prefix.'-'.time().".".$image->getClientOriginalExtension();
        $image_path='uploads/'.$path;
        $image->move(public_path($image_path),$image_name);
        return $image_path.'/'.$image_name;
    }
 </pre>
Then you create a Resource controller
Your command like:

```
php artisan make:model BrandController -mcr
```


## Store Brand code 
```
/**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // for validation
        $request->validate([
            'brand_name'=>'required|unique:brands,brand_name',

        ]); //end validation

        // for brand data
        $brandData=array(
            'brand_name'=>$request->brand_name,
            'brand_slug'=>Str::slug($request->brand_name,'-'),
        ); //end brand data

        // if brand has image file
        if($request->hasFile('brand_image')){
            $brandData['brand_image']= $this->uploadImage('B',$request->brand_image,'brand');;
        }//end brand image

        // for insert brand
        $brand=Brand::create($brandData);
        //end insert

         // brand insert success or fail message and redirect
        if($brand){
            Session::flash('success',"Successfully Add  Brand");
            return redirect()->back();
       }else{
           Session::flash('error',"Failed Add Brand  ");
            return redirect()->back();
       } // // end  brand insert success or fail message and redirect

    }
```

## Update Brand Code 
```
/**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Brand  $brand
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Brand $brand)
    {
        // for validattion
        $request->validate([
            'brand_name'=>'required|unique:brands,brand_name,'.$brand->id,

        ]); //end validation

        // for brand data
        $brandData=array(
            'brand_name'=>$request->brand_name,
            'brand_slug'=>Str::slug($request->brand_name,'-'),

        ); //end brand data

        //if brand has image file
        if($request->hasFile('brand_image')){
            if(isset($brand->brand_image) && file_exists(public_path($brand->brand_image))){
                unlink(public_path($brand->brand_image));
            }
            $brandData['brand_image']= $this->uploadImage('B',$request->brand_image,'brand');;
        }//end image insert

        //for update brand
        $brand->update($brandData);

        // brand update success or fail message and redirect
        if($brand){
            Session::flash('success',"Successfully Update  Brand");
            return redirect()->back();
        }else{
           Session::flash('error',"Failed Update Brand  ");
            return redirect()->back();
        } //end brand update success or fail message and redirect
    }
```

## Delete Brand Code
```
 /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Brand  $brand
     * @return \Illuminate\Http\Response
     */
    public function destroy(Brand $brand)
    {
        // for check brand image filed and delete brand with image
        if(!empty($brand->image) && file_exists(public_path($brand->image) )){
            unlink(public_path($brand->image));
            $delete=$brand->delete();
        }else{
            $delete=$brand->delete();
        }

        //for success or fail message
        if($delete){
            return response()->json(['success'=>"Successfully Delete Brand"]);
        }else{
            return response()->json(['error'=>"Failed To Delete Brand"],500);
        }//end success or fail message

        return redirect()->back();
    }
```

