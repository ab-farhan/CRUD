# All CRUD Here
## Resource CURD (-mcr) 
###### Here we create custom image upload
 First of all you need to put this bellow code  **Base Controloler**
```
  public function uploadImage($prefix,$image,$path){
        $image_name=$prefix.'-'.time().".".$image->getClientOriginalExtension();
        $image_path='uploads/'.$path;
        $image->move(public_path($image_path),$image_name);
        return $image_path.'/'.$image_name;
    }
```
 
###### Then you create a Resource controller
###### Your command like:
```
php artisan make:model BrandController -mcr
```
**Copy the bellow code and paste your controller, change controller name, model name and others you need**

```
<?php

namespace App\Http\Controllers;

use App\Models\Brand;
use Illuminate\Http\Request;
use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\Str;

class BrandController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $brands=Brand::orderby('id','DESC')->get();
        return view('brand.all',compact('brands'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('brand.add');
    }

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

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Brand  $brand
     * @return \Illuminate\Http\Response
     */
    public function show(Brand $brand)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Models\Brand  $brand
     * @return \Illuminate\Http\Response
     */
    public function edit(Brand $brand)
    {
        return view('brand.edit',compact('brand'));
    }

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
}
```

