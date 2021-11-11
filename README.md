# All CRUD Here
## 1. Resource CURD (-mcr) 
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
Image Upload with **Image Intervention**
```
use Intervention\Image\Facades\Image;
```
```
public function uploadImage($prefix,$image,$path){

        $image_name=$prefix.'-'.time().".".$image->getClientOriginalExtension();
        $image_path='uploads/'.$path.'/'.$image_name;
        // $image->move(public_path($image_path),$image_name);
        Image::make($image)->save('uploads/'.$path.'/'.$image_name);
        // return $image_path.'/'.$image_name;
        return $image_path;
    }
```
###### Image Upload with **SIZE**
```
public function uploadImageSize($prefix,$image,$path,$width,$height){

        $image_name=$prefix.'-'.time().".".$image->getClientOriginalExtension();
        $image_path='uploads/'.$path.'/'.$image_name;
        Image::make($image)->resize($width,$height)->save('uploads/'.$path.'/'.$image_name);
        return $image_path;
    }
```
## you neeed to use this code your controller
```
    if($request->hasFile('brand_image')){
        $brandData['brand_image']= $this->uploadImage('prefix',$request->image_name,'Floder_Name');
    }//end brand image
```
###### with image size
 ```
 if($request->hasFile('member_image')){
            $userData['member_image'] = $this->uploadImageSize('user',$request->member_image,'user',150,160);
        }
 ```
 ## Delete With sweet alet2 code
 ```
 <script>
    $(document).ready(function(){
        $(".deleteBtn").click(function(e) {
          let  url = $(this).attr('data-url');
            e.preventDefault()
            Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
                if (result.isConfirmed) {
                    $.ajax({
                            url: url,
                            type: 'DELETE',
                            dataType : 'json',
                            success : function(res){
                                Swal.fire( 'Success!', 'Your file has been deleted.', 'success')
                                setTimeout(function(){
                                    window.location.reload(1);
                                }, 2000);
                            },
                            error : function(res){
                                Swal.fire( 'Failed!', 'Somethung went wrong.', 'error')
                            },
                    });
                } else {
                    Swal.fire('Safe Now!','Your imaginary file is safe :)', 'info')
                }
            });
        });

    });

</script>
 ```
###### Then you create a Resource controller
###### Your command like:
```
php artisan make:model Brand -mcr
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
## 2. Resource CURD (-r) 

**Before copy images code and paste it _BASE CONTROLLER_**

###### Your create a Resource controller
###### Your command like:
```
php artisan make:controller WarehouseStaffController -r
```
**Copy the bellow code and paste your controller, change controller name, model name and others you need**
```
<?php

namespace App\Http\Controllers;

use App\Models\WarehouseStaff;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Session;

class WarehouseStaffController extends Controller
{
    public function indexView(){
        return view('warehouse_staff.index');
    }

    public function login(Request $request){
        $request->validate([
            'email'=>['required','email','exists:warehouse_staff,email'],
            'password'=>['required','min:8'],
        ]);

        $creds=$request->only('email','password');
        if(Auth::guard('warehousestaff')->attempt($creds)){
            return redirect()->route('warehousestaff.home');
        }else{
            return redirect()->route('warehousestaff.login')->with('error',"Failed To login  Warehouse Staff");
        }
    }

    public function logout(){
        Auth::guard('warehousestaff')->logout();
        return redirect()->route('warehousestaff.login');
    }


     /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $users=WarehouseStaff::orderBy('id','DESC')->get();
        return view('warehouse_staff.all',compact('users'));
    }

    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('warehouse_staff.add');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {

        $request->validate([
            'name'=>'required|min:3',
            'date_of_birth'=>'required',
            'phone'=>'required|min:11|max:11',
            'nid_number'=>'required',
            'location'=>'required',
            'employee_office_id'=>'required',
            'email'=>'required|email|unique:warehouse_staff,email',
            'password'=>'required|min:8',
        ]);

        if($request->hasFile('image')){
            $image=$request->file('image');
            $image_name='U-'.time().".".$image->getClientOriginalExtension();
            $image_path='uploads/warehousestaff/'.$image_name;
            $image->move(public_path('uploads/warehousestaff'),$image_name);

            $user=WarehouseStaff::create([
                'name'=>$request->name,
                'warehouse_staff_image'=>$image_path,
                'description'=>$request->description,
                'location'=>$request->location,
                'department_id'=>$request->department_id,
                'father_name'=>$request->father_name,
                'mother_name'=>$request->mother_name,
                'permanent_address'=>$request->email,
                'persent_address'=>$request->present_address,
                'permanent_address'=>$request->permanent_address,
                'mobile_nember'=>$request->phone,
                'nid_number'=>$request->nid_number,
                'date_of_birth'=>$request->date_of_birth,
                'employee_office_id'=>$request->employee_office_id,
                'email'=>$request->email,
                'password'=>Hash::make($request->password),
                'created_at'=>Carbon::now()->toDateTimeString(),

            ]);
        }else{
            $user=WarehouseStaff::create([
                'name'=>$request->name,
                'description'=>$request->description,
                'location'=>$request->location,
                'department_id'=>$request->department_id,
                'father_name'=>$request->father_name,
                'mother_name'=>$request->mother_name,
                'permanent_address'=>$request->email,
                'persent_address'=>$request->present_address,
                'permanent_address'=>$request->permanent_address,
                'mobile_nember'=>$request->phone,
                'nid_number'=>$request->nid_number,
                'date_of_birth'=>$request->date_of_birth,
                'employee_office_id'=>$request->employee_office_id,
                'email'=>$request->email,
                'password'=>Hash::make($request->password),
                'created_at'=>Carbon::now()->toDateTimeString(),

            ]);
        }

        if($user){
            Session::flash('success',"Successfully Create Delivary Boy");
            return redirect()->back();
       }else{
           Session::flash('error',"Failed Create Delivary Boy");
            return redirect()->back();
       }

    }

    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function edit($id)
    {
        $user=WarehouseStaff::where('id',$id)->first();
        return view('warehouse_staff.edit',compact('user'));
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        $request->validate([
            'name'=>'required|min:3',
            'date_of_birth'=>'required',
            'phone'=>'required|min:11|max:11',
            'nid_number'=>'required',
            'location'=>'required',
            'email'=>'required|email|unique:warehouse_staff,email,'.$id,
        ]);

        $finduser=WarehouseStaff::findOrFail($id);

        if($request->hasFile('image')){
            if(file_exists(public_path($finduser->warehouse_staff_image))){
                unlink(public_path($finduser->warehouse_staff_image));
            }
            $image=$request->file('image');
            $image_name='U-'.time().".".$image->getClientOriginalExtension();
            $image_path='uploads/warehousestaff/'.$image_name;
            $image->move(public_path('uploads/warehousestaff'),$image_name);

            $user=WarehouseStaff::where('id',$id)->update([
                'name'=>$request->name,
                'warehouse_staff_image'=>$image_path,
                'description'=>$request->description,
                'location'=>$request->location,
                'department_id'=>$request->department_id,
                'father_name'=>$request->father_name,
                'mother_name'=>$request->mother_name,
                'permanent_address'=>$request->email,
                'persent_address'=>$request->present_address,
                'permanent_address'=>$request->permanent_address,
                'mobile_nember'=>$request->phone,
                'nid_number'=>$request->nid_number,
                'date_of_birth'=>$request->date_of_birth,
                'email'=>$request->email,
                'updated_at'=>Carbon::now()->toDateTimeString(),

            ]);
        }else{
            $user=WarehouseStaff::where('id',$id)->update([
                'name'=>$request->name,
                'description'=>$request->description,
                'location'=>$request->location,
                'department_id'=>$request->department_id,
                'father_name'=>$request->father_name,
                'mother_name'=>$request->mother_name,
                'permanent_address'=>$request->email,
                'persent_address'=>$request->present_address,
                'permanent_address'=>$request->permanent_address,
                'mobile_nember'=>$request->phone,
                'nid_number'=>$request->nid_number,
                'date_of_birth'=>$request->date_of_birth,
                'email'=>$request->email,
                'updated_at'=>Carbon::now()->toDateTimeString(),

            ]);
        }

        if($user){
            Session::flash('success',"Successfully Update Warehouse Staff");
            return redirect()->back();
       }else{
           Session::flash('error',"Failed Update Warehouse Staff");
            return redirect()->back();
       }
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy($id)
    {
        $user=WarehouseStaff::findOrFail($id);


        if(!empty($user->warehouse_staff_image) && file_exists(public_path($user->warehouse_staff_image) )){
               unlink(public_path($user->warehouse_staff_image));
               $delete=$user->delete();
        }else{
           $delete=$user->delete();
        }

        if($delete){
           return response()->json(['success'=>"Successfully Delete Warehouse Staff"]);
        }else{
           return response()->json(['error'=>"Failed Delete Warehouse Staff"],500);
        }
        return redirect()->back();
   }



}

```
## 3. Normal CURD  

**Before copy images code and paste it _BASE CONTROLLER_**

###### Your Create a Normal Controller
###### Your command like:
```
php artisan make:controller WarehouseStaffController 
```
**Copy the bellow code and paste your controller, change controller name, model name and others you need**

```
<?php

namespace App\Http\Controllers;

use App\Models\Admin;
use Illuminate\Http\Request;
use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Session;
use Intervention\Image\Facades\Image;

use function PHPUnit\Framework\fileExists;

class AdminController extends Controller
{
    // for showing admin page
    public function index(){
        return view('admin.index');
    }
    // for showing all admins
    public function allAdmin(){
        $admins=Admin::get();
        return view('admin.all',compact('admins'));
    }
    // for admin create form page show
    public function create(){
        return view('admin.add');
    }



    //================= for create admin===================
    public function store(Request $request){
        // dd($request->all());
        //for validtion
        $request->validate([
            'name'=>'required|min:3',
            'phone'=>'nullable|min:11|max:11',
            'email'=>'required|email|unique:admins,email',
            'password'=>'required|min:8|confirmed',
        ]);//end validation

        //for admin data
        $adminData=array(
            'name'=>$request->name,
            'phone'=>$request->phone,
            'email'=>$request->email,
            'password'=>Hash::make($request->password),
        );//end admin data

        //for check image file
        if($request->hasFile('image')){
            $adminData['image']=$this->uploadImage('A',$request->image,'admin');
        }//end check image file

        $admin=Admin::create($adminData);

        // for fail ,success & redirect message
        if($admin){
             Session::flash('success',"Successfully create admin");
             return redirect()->back();
        }else{
            Session::flash('error',"Failed create admin");
             return redirect()->back();
        }//end  fail ,success & redirect message

    } //===========end store===================

    //=====================  for edit admin==================
    public function edit($id){
        $admin=Admin::findOrFail($id);
        return view('admin.edit',compact('admin'));
    }//======================end edit admin=====================

    //  for update admin
public function update(Request $request, $id){

    // for validation
        $request->validate([
            'name'=>'required',
            'phone'=>'nullable|min:11|max:11',
            'email'=>'required|email|unique:admins,email,'.$id,
        ]);//end validation

        // for find admin
        $findadmin=Admin::findOrFail($id);

        //for admin data
        $adminData=array(
            'name'=>$request->name,
            'phone'=>$request->phone,
            'email'=>$request->email,
            'updated_at'=>Carbon::now()->toDateTimeString(),
        );//end admin data

        // for check image file
        if($request->hasFile('image')){

            if(file_exists(public_path($findadmin->image))){
                unlink(public_path($findadmin->image));
            }

            $adminData['image']=$this->uploadImage('A',$request->image,'admin');

        }//end check image

        // for update admin
        $admin=Admin::where('id',$id)->update($adminData);
        //end update admin

        //for success or fail message
        if($admin){
            Session::flash('success',"Successfully Update admin");

       }else{
           Session::flash('error',"Failed Update admin");

       }//end success or fail message
       return redirect()->back();
}// ============================ end  update =========================

    // ================== for delete admin ==============================
    public function destroy( $id){
        $admin=Admin::findOrFail($id);

        // for check image file exist, if exists delete it
         if(!empty($admin->image) && file_exists(public_path($admin->image) )){
                unlink(public_path($admin->image));
                $delete=$admin->delete();
         }else{
            $delete=$admin->delete();
         }//end check image and delete

         // for success , fail message
         if($delete){
            return response()->json(['success'=>"Successfully Delete Admins"]);
         }else{
            return response()->json(['error'=>"Failed Delete Admins"],500);
         }//end success faiul message
         return redirect()->back();

    }//================end destroy admin================



    //====================== for admin login ======================
    public function login(Request $request){
        $request->validate([
            'email'=>['required','email','exists:admins,email'],
            'password'=>['required','min:8'],
        ]);

        $creds=$request->only('email','password');
        if(Auth::guard('admin')->attempt($creds)){
            return redirect()->route('admin.home');
        }else{
            return redirect()->route('admin.login')->with('error',"Failed To login");
        }

    }// ===================  end login  ============================


    //===================== for logout admin  ====================
    public function logout(){
        Auth::guard('admin')->logout();
        return redirect()->route('admin.home');

    }//==================  end logout ===============
}
```

