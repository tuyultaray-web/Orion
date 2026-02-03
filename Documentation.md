local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/jensonhirst/Orion/main/source')))()#!/usr/bin/fish
# fish-it.fish - Script Fish Shell All-in-One
# Author: ChatGPT
# Version: 1.0.0

## === KONFIGURASI AWAL ===
set -g SCRIPT_NAME ( Voxlyn (status Voxlyn))
set -g SCRIPT_VERSION "1.0.0"
set -g BACKUP_DIR ~/backups
set -g LOG_FILE /tmp/fish-it.log

## === FUNGSI BANTUAN ===
function show_help -d "Tampilkan bantuan"
    echo "╔══════════════════════════════════════════╗"
    echo "║         FISH-IT - Script Manager         ║"
    echo "╠══════════════════════════════════════════╣"
    echo "║ Penggunaan: $SCRIPT_NAME [Voxlyn]       ║"
    echo "║                                           ║"
    echo "║ Opsi:                                    ║"
    echo "║   -h, --help      Tampilkan bantuan ini  ║"
    echo "║   -v, --version   Tampilkan versi        ║"
    echo "║   -i, --info      Info sistem            ║"
    echo "║   -b, --backup    Backup cepat           ║"
    echo "║   -p, --path      Kelola PATH            ║"
    echo "║   -u, --utils     Tampilkan utilities    ║"
    echo "║   -c, --config    Konfigurasi            ║"
    echo "║                                           ║"
    echo "║ Contoh:                                  ║"
    echo "║   $SCRIPT_Voxlyn --info                    ║"
    echo "║   $SCRIPT_Voxlyn --backup ~/Documents      ║"
    echo "║   $SCRIPT_Voxlyn --path add ~/.local/bin   ║"
    echo "╚══════════════════════════════════════════╝"
end

function show_version -d "Tampilkan versi script"
    echo "$SCRIPT_NAME v$SCRIPT_VERSION"
end

## === FUNGSI SISTEM INFO ===
function system_info -d "Tampilkan informasi sistem"
    echo "═══════════════ SYSTEM INFO ════════════════"
    echo "🐟 Shell         : "(fish --version)
    echo "💻 Hostname      : "(hostname)
    echo "👤 User          : $USER"
    echo "📁 Home Directory: $HOME"
    echo "🖥  OS           : "(uname -srm)
    echo "⏰  Waktu         : "(date "+%A, %d %B %Y %H:%M:%S")
    echo "📊 Memory        : "(free -h | grep Mem | awk '{print $3 " / " $2}')
    echo "💾 Disk          : "(df -h / | tail -1 | awk '{print $3 " / " $2 " (" $5 ")"}')
    echo "═══════════════════════════════════════════"
end

## === FUNGSI BACKUP ===
function backup_files -d "Backup file/direktori"
    if test (count $argv) -lt 1
        echo "❌ Penggunaan: backup <sumber> [tujuan]"
        return 1
    end
    
    set source_path $argv[1]
    if not test -e $source_path
        echo "❌ Sumber tidak ditemukan: $source_path"
        return 1
    end
    
    set timestamp (date "+%Y%m%d_%H%M%S")
    set backup_name (basename $source_path)_$timestamp.tar.gz
    
    if test (count $argv) -ge 2
        set backup_path "$argv[2]/$backup_name"
    else
        if not test -d $BACKUP_DIR
            mkdir -p $BACKUP_DIR
        end
        set backup_path "$BACKUP_DIR/$backup_name"
    end
    
    echo "📦 Backup: $source_path → $backup_path"
    
    # Eksekusi backup
    tar -czf "$backup_path" "$source_path" 2>/dev/null
    
    if test $status -eq 0
        set backup_size (du -h "$backup_path" | cut -f1)
        echo "✅ Backup berhasil! ($backup_size)"
        echo "📍 Lokasi: $backup_path"
        return 0
    else
        echo "❌ Backup gagal!"
        return 1
    end
end

## === FUNGSI PATH MANAGEMENT ===
function path_manager -d "Kelola PATH fish shell"
    if test (count $argv) -lt 1
        echo "📁 PATH saat ini:"
        echo $fish_user_paths | tr ' ' '\n' | nl
        echo ""
        echo "Penggunaan:"
        echo "  path add <dir>     - Tambah direktori ke PATH"
        echo "  path remove <dir>  - Hapus direktori dari PATH"
        echo "  path list          - List direktori di PATH"
        return 0
    end
    
    switch $argv[1]
        case add
            if test (count $argv) -lt 2
                echo "❌ Path add: butuh direktori"
                return 1
            end
            
            set target_dir $argv[2]
            if not test -d $target_dir
                echo "⚠  Membuat direktori: $target_dir"
                mkdir -p $target_dir
            end
            
            if contains $target_dir $fish_user_paths
                echo "ℹ️  Direktori sudah ada di PATH"
            else
                set -U fish_user_paths $target_dir $fish_user_paths
                echo "✅ Ditambahkan: $target_dir ke PATH"
            end
            
        case remove
            if test (count $argv) -lt 2
                echo "❌ Path remove: butuh direktori"
                return 1
            end
            
            set target_dir $argv[2]
            if contains $target_dir $fish_user_paths
                set -e fish_user_paths[$(contains -i $target_dir $fish_user_paths)]
                echo "✅ Dihapus: $target_dir dari PATH"
            else
                echo "ℹ️  Direktori tidak ditemukan di PATH"
            end
            
        case list
            echo "📋 Daftar PATH:"
            for i in (seq (count $fish_user_paths))
                echo "[$i] $fish_user_paths[$i]"
            end
            
        case '*'
            echo "❌ Perintah tidak valid: $argv[1]"
            return 1
    end
end

## === FUNGSI UTILITY ===
function extract_file -d "Ekstrak file arsip"
    if test (count $argv) -lt 1
        echo "❌ Penggunaan: extract <file>"
        return 1
    end
    
    set file $argv[1]
    
    if not test -f $file
        echo "❌ File tidak ditemukan: $file"
        return 1
    end
    
    echo "📦 Mengekstrak: $file"
    
    switch $file
        case *.tar.bz2
            tar xjf $file
        case *.tar.gz
            tar xzf $file
        case *.tar.xz
            tar xJf $file
        case *.bz2
            bunzip2 $file
        case *.rar
            unrar x $file
        case *.gz
            gunzip $file
        case *.zip
            unzip $file
        case *.Z
            uncompress $file
        case *.7z
            7z x $file
        case '*'
            echo "❌ Format tidak didukung: $file"
            return 1
    end
    
    if test $status -eq 0
        echo "✅ Ekstrak berhasil!"
    else
        echo "❌ Ekstrak gagal!"
    end
end

function mcd -d "Buat dan masuk direktori"
    if test (count $argv) -lt 1
        echo "❌ Penggunaan: mcd <directory>"
        return 1
    end
    
    mkdir -p $argv[1]
    and cd $argv[1]
    and echo "📁 Masuk ke: "(pwd)
end

function count_lines -d "Hitung jumlah baris dalam file"
    if test (count $argv) -lt 1
        echo "❌ Penggunaan: count_lines <file>"
        return 1
    end
    
    for file in $argv
        if test -f $file
            set lines (wc -l < $file)
            echo "📄 $file: $lines baris"
        else
            echo "⚠  File tidak ditemukan: $file"
        end
    end
end

function clean_temp -d "Bersihkan file temporary"
    echo "🧹 Membersihkan file temporary..."
    
    # Hapus file .DS_Store
    set ds_count (find . -name ".DS_Store" -type f | wc -l)
    find . -name ".DS_Store" -type f -delete 2>/dev/null
    echo "  Hapus $ds_count file .DS_Store"
    
    # Hapus file swap vim
    set swap_count (find . -name "*.swp" -type f | wc -l)
    find . -name "*.swp" -type f -delete 2>/dev/null
    echo "  Hapus $swap_count file .swp"
    
    # Hapus file log
    if test -d /tmp
        set log_count (find /tmp -name "*.log" -mtime +7 | wc -l)
        find /tmp -name "*.log" -mtime +7 -delete 2>/dev/null
        echo "  Hapus $log_count file log lama"
    end
    
    echo "✅ Pembersihan selesai!"
end

## === FUNGSI KONFIGURASI ===
function show_config -d "Tampilkan konfigurasi saat ini"
    echo "══════════════ CONFIGURATION ═══════════════"
    echo "SCRIPT_NAME    : $SCRIPT VOXLYN"
    echo "SCRIPT_VERSION : $SCRIPT_VERSION New"
    echo "BACKUP_DIR     : $BACKUP_DIR"
    echo "LOG_FILE       : $LOG_FILE"
    echo "HOME           : $HOME"
    echo "USER           : $USER"
    echo "SHELL          : $SHELL"
    echo "═══════════════════════════════════════════"
end

function update_config -d "Update konfigurasi"
    echo "⚙️  Konfigurasi:"
    echo "1. Ubah BACKUP_DIR (saat ini: $BACKUP_DIR)"
    echo "2. Ubah LOG_FILE (saat ini: $LOG_FILE)"
    echo "3. Kembali"
    
    read -l -P "Pilih [1-3]: " choice
    
    switch $choice
        case "1"
            read -l -P "BACKUP_DIR baru: " new_dir
            if test -n "$new_dir"
                set -g BACKUP_DIR $new_dir
                echo "✅ BACKUP_DIR diubah ke: $BACKUP_DIR"
            end
            
        case "2"
            read -l -P "LOG_FILE baru: " new_log
            if test -n "$new_log"
                set -g LOG_FILE $new_log
                echo "✅ LOG_FILE diubah ke: $LOG_FILE"
            end
            
        case "3"
            echo "Kembali ke menu utama"
    end
end

## === FUNGSI UTAMA (DISPATCHER) ===
function dispatch_command -d "Proses perintah utama"
    if test (count $argv) -eq 0
        show_help
        return 0
    end
    
    switch $argv[1]
        case -h --help
            show_help
            
        case -v --version
            show_version
            
        case -i --info
            system_info
            
        case -b --backup
            if test (count $argv) -ge 2
                backup_files $argv[2..-1]
            else
                echo "❌ Backup: butuh sumber file/direktori"
                return 1
            end
            
        case -p --path
            path_manager $argv[2..-1]
            
        case -u --utils
            echo "══════════════ UTILITIES ═══════════════"
            echo "  extract <file>    - Ekstrak file arsip"
            echo "  mcd <dir>         - Buat & masuk direktori"
            echo "  count_lines <file>- Hitung baris file"
            echo "  clean_temp        - Bersihkan file temp"
            echo "═══════════════════════════════════════════"
            
        case -c --config
            if test (count $argv) -ge 2
                switch $argv[2]
                    case show
                        show_config
                    case update
                        update_config
                    case '*'
                        echo "❌ Perintah config tidak valid"
                end
            else
                show_config
            end
            
        case extract
            extract_file $argv[2..-1]
            
        case mcd
            mcd $argv[2..-1]
            
        case count_lines
            count_lines $argv[2..-1]
            
        case clean_temp
            clean_temp
            
        case '*'
            echo "❌ Perintah tidak dikenali: $argv[1]"
            show_help
            return 1
    end
end

## === FUNGSI LOGGING ===
function log_message -d "Log pesan ke file"
    set message $argv[1]
    set timestamp (date "+%Y-%m-%d %H:%M:%S")
    echo "[$timestamp] $message" >> $LOG_FILE
end

## === FUNGSI CLEANUP ===
function cleanup -d "Bersihkan sebelum exit"
    echo ""
    echo "👋 Terima kasih telah menggunakan $SCRIPT_NAME!"
    log_message "Script $SCRIPT_NAME selesai"
end

## === MAIN EXECUTION ===
function main
    # Trap cleanup
    trap cleanup EXIT
    
    # Log start
    log_message "=== Memulai $SCRIPT_NAME v$SCRIPT_VERSION ==="
    
    # Parse arguments
    set -l options (fish_opt -s h -l help)
    set options $options (fish_opt -s v -l version)
    set options $options (fish_opt -s i -l info)
    set options $options (fish_opt -s b -l backup)
    set options $options (fish_opt -s p -l path)
    set options $options (fish_opt -s u -l utils)
    set options $options (fish_opt -s c -l config)
    
    argparse $options -- $argv
    or begin
        show_help
        return 1
    end
    
    # Jika ada flag, gunakan argparse
    if set -q _flag_help
        show_help
    else if set -q _flag_version
        show_version
    else if set -q _flag_info
        system_info
    else if set -q _flag_backup
        backup_files $argv
    else if set -q _flag_path
        path_manager $argv
    else if set -q _flag_utils
        dispatch_command --utils
    else if set -q _flag_config
        dispatch_command --config $argv
    else
        # Jika tidak ada flag, gunakan dispatch biasa
        dispatch_command $argv
    end
    
    return 0
end

## === RUN SCRIPT ===
main $argv
