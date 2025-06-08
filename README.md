Nama	Muhammad Nurul Firdaus
Kelas	TI.23.A.5
Mata Kuliah	Pemrograman Web 2
Langkah-langkah Praktikum
Praktikum 4: Membuat Sistem Login
1. Basis Data Persiapkan
Buat tabel userpada database dengan SQL berikut:

CREATE TABLE user (
  id INT(11) auto_increment,
  username VARCHAR(200) NOT NULL,
  useremail VARCHAR(200),
  userpassword VARCHAR(200),
  PRIMARY KEY(id)
);
teks alternatif

2. Membuat Model Pengguna
Buat model UserModel.phppada direktori app/Models:

<?php
namespace App\Models;
use CodeIgniter\Model;

class UserModel extends Model
{
    protected $table = 'user';
    protected $primaryKey = 'id';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['username', 'useremail', 'userpassword'];
}
3. Membuat Controller Pengguna
Buat pengontrol User.phpdengan metode index()dan login()untuk mengelola pengguna dan login:

<?php

namespace App\Controllers;

use App\Models\UserModel;

class User extends BaseController
{
    public function index()
    {
        $title = 'Daftar User';
        $model = new UserModel();
        $users = $model->findAll();
        return view('user/index', compact('users', 'title'));
    }
    public function login()
    {
        helper(['form']);
        $email = $this->request->getPost('email');
        $password = $this->request->getPost('password');
        if (!$email)
        {
        return view('user/login');
        }

        $session = session();
        $model = new UserModel();
        $login = $model->where('useremail', $email)->first();
        if ($login)
        {
            $pass = $login['userpassword'];
            if (password_verify($password, $pass))
            {
                $login_data = [
                'user_id' => $login['id'],
                'user_name' => $login['username'],
                'user_email' => $login['useremail'],
                'logged_in' => TRUE,
                ];

                $session->set($login_data);
                return redirect('admin/artikel');
            }
            else
            {
                $session->setFlashdata("flash_msg", "Password salah.");
                return redirect()->to('/user/login');
            }
        }
        else
        {
            $session->setFlashdata("flash_msg", "email tidak terdaftar.");
            return redirect()->to('/user/login');
        }
    }
}
4. Membuat Tampilan Login
Buat tampilan login.phpuntuk form login:

<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<title>Login</title>
		<link rel="stylesheet" href="<?= base_url('/style.css'); ?>" />
	</head>

	<body>
		<div id="login-wrapper">
			<h1>Sign In</h1>
			<?php if (session()->getFlashdata('flash_msg')): ?>
			<div class="alert alert-danger">
				<?= session()->getFlashdata('flash_msg') ?>
			</div>
			<?php endif; ?>
			<form action="" method="post">
				<div class="mb-3">
					<label for="InputForEmail" class="form-label">Email address</label>
					<input
						type="email"
						name="email"
						class="form-control"
						id="InputForEmail"
						value="<?= set_value('email') ?>"
					/>
				</div>
				<div class="mb-3">
					<label for="InputForPassword" class="form-label">Password</label>

					<input
						type="password"
						name="password"
						class="form-control"
						id="InputForPassword"
					/>
				</div>
				<button type="submit" class="btn btn-primary">Login</button>
			</form>
		</div>
	</body>
</html>
5. Membuat Seeder Database
Database seeder digunakan untuk membuat data dummy. Untuk keperluan ujicoba modul login, kita perlu memasukkan data user dan password kedaalam database. Untuk itu membuat database seeder untuk tabel user. Buka CLI, kemudian tulis perintah berikut:

php spark make:seeder UserSeeder
Selanjutnya, buka file UserSeeder.php yang berada di lokasi direktori/app/Database/Seeds/UserSeeder.php kemudian isi dengan kode berikut:

<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run()
    {
        $model = model('UserModel');
        $model->insert([
            'username' => 'admin',
            'useremail' => 'admin@email.com',
            'userpassword' => password_hash('admin123', PASSWORD_DEFAULT),
        ]);
    }
}
Selanjutnya buka kembali CLI dan ketik perintah berikut:

php spark db:seed UserSeeder
Uji Coba Login
teks alternatif

6. Membuat Filter Auth
Buat filter Auth.phpuntuk membatasi akses ke halaman admin:

<?php namespace App\Filters;

use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\Filters\FilterInterface;

class Auth implements FilterInterface
{
    public function before(RequestInterface $request, $arguments = null)
    {
        // jika user belum login
        if (! session()->get('logged_in')) {
            // maka redirct ke halaman login
            return redirect()->to('/user/login');
        }
    }
    public function after(RequestInterface $request, ResponseInterface $response, $arguments = null)
    {
        // Do something here
    }
}
Selanjutnya buka file app/Config/Filters.php tambahkan kode berikut:

'auth' => App\Filters\Auth::class
teks alternatif Selanjutnya buka file app/Config/Routes.php dan sesuaikan kodenya. teks alternatif

7. Coba Akses Menu Admin
Buka url dengan alamat http://localhost:8080/admin/artikel ketika alamat tersebut diakses maka, akan muncul halaman login. teks alternatif

8. Fungsi Logout
Tambahkan metode logout pada Controller User seperti berikut:

public function logout()
    {
        session()->destroy();
        return redirect()->to('/user/login');
    }
Praktikum 5: Paginasi dan Pencarian
1. Membuat Pagination
Modifikasi artikel pengontrol untuk menambahkan pagination:

public function admin_index()
{
    $title = 'Daftar Artikel';
    $model = new ArtikelModel();
    $data = [
        'title' => $title,
        'artikel' => $model->paginate(10), #data dibatasi 10 record per halaman
        'pager' => $model->pager,
    ];
    return view('artikel/admin_index', $data);
}
Kemudian buka file views/artikel/admin_index.php dan tambahkan kode berikut dibawah deklarasi data tabel.

<?= $pager->links(); ?>
Selanjutnya buka kembali menu daftar artikel, tambahkan data lagi untuk melihat hasilnya. teks alternatif

2. Membuat Pencarian
Modifikasi pengontrol untuk menambahkan data pencarian:

public function admin_index()
    {
        $title = 'Daftar Artikel';
        $q = $this->request->getVar('q') ?? '';
        $model = new ArtikelModel();
        $data = [
            'title' => $title,
            'q' => $q,
            'artikel' => $model->like('judul', $q)->paginate(10), # data dibatasi 10 record per halaman
            'pager' => $model->pager,
        ];
        return view('artikel/admin_index', $data);
    }
Kemudian buka kembali file views/artikel/admin_index.php dan tambahkan form pencarian sebelum deklarasi tabel seperti berikut:

<form method="get" class="form-search">
    <input type="text" name="q" value="<?= $q; ?>" placeholder="Cari data">
    <input type="submit" value="Cari" class="btn btn-primary">
</form>
Dan pada link pager ubah seperti berikut.

<?= $pager->only(['q'])->links(); ?>
3. Uji Coba Pagination dan Pencarian
Selanjutnya ujicoba dengan membuka kembali halaman admin artikel, masukkan kata kunci tertentu pada form pencarian. teks alternatif

Praktikum 6: Upload File Gambar
1. Artikel Modifikasi Kontroler
Buka Kembali Controller Artikel pada project sebelumnya, sesuaikan kode pada method add seperti berikut:

public function add()
    {
        // validasi data.
        $validation = \Config\Services::validation();
        $validation->setRules(['judul' => 'required']);
        $isDataValid = $validation->withRequest($this->request)->run();
        if ($isDataValid) {
            $file = $this->request->getFile('gambar');
            $file->move(ROOTPATH . 'public/gambar');
            $artikel = new ArtikelModel();
            $artikel->insert([
                'judul' => $this->request->getPost('judul'),
                'isi' => $this->request->getPost('isi'),
                'slug' => url_title($this->request->getPost('judul')),
                'gambar' => $file->getName(),
            ]);
            return redirect('admin/artikel');
        }
        $title = "Tambah Artikel";
        return view('artikel/form_add', compact('title'));
    }
2. Modifikasi Lihat Artikel
Tambahkan field input file pada form artikel:

<p>
    <input type="file" name="gambar">
</p>
Dan sesuaikan tag form dengan menambahkan ecrypt type seperti berikut.

<form action="" method="post" enctype="multipart/form-data">
3. Uji Coba Upload Gambar
Akses menu tambah artikel dan uji coba upload gambar. teks alternatif
Laporan Praktikum
Pastikan untuk screenshot setiap perubahan yang dilakukan pada setiap langkah praktikum.
Update file README.mddan tuliskan penjelasan serta screenshot dari setiap langkah praktikum.
Komit hasilnya pada repositori dan kirimkan URL repositori ke e-learning.
