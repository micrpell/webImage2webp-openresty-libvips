# webImage2webp-openresty-libvips
an extensible method for convering website images to webp.

一种可扩展的方法，将网站图像转换为webp格式。

## 环境
1. web平台为openresty1.27.1.2及其以下版本

    https://openresty.org/en/ann-1027001002.html

2. 有着libvips库

    https://www.libvips.org/


## 代码

--------分离链接orig_path与变体variant--------
local orig_path, variant = ngx.var.uri:match("^(/photo/.+%.%w+)([!_%-].*)$")


--------剔除不进行转换的变体--------

-- 1.无变体或为空则直接返回
if not variant or variant == "" then
    local args = ngx.var.args
    -- 如果参数存在且只包含安全字符，则将其视为变体
    if args and args:match("^[a-zA-Z0-9/_]+$") then
        variant = "!" .. args
    else
        return ngx.exec(uri)
    end
end

-- 2.变体仅 "!" 或 "_"，直接返回
if variant == "!" or variant == "_" then
    return ngx.exec(orig_path)
end


--------定义相关路径--------

-- 1.确保文件路径安全
local safe_variant = variant:gsub("/", "_")
-- 2.网站根路径，此处替换为你设定的
local root_dir = "/var/www/img" 
-- 3.原图片物理路径
local src = root_dir .. orig_path
-- 4.生成图物理路径，需要你创建这个文件夹“/cache”
local cache_path = root_dir .. "/cache" .. orig_path .. safe_variant .. ".webp"
-- 5.返回路径
local return_path = "/cache" .. orig_path .. safe_variant .. ".webp"

--------测试文件是否已存在--------

local f = io.open(cache_path, "rb")
if f then
    f:close()
    return ngx.exec(return_path)
end

--------图片操作--------
-- 如果需要扩展用法，在这里添加if语句就可以了

if variant == "!2webp" then
    local quality = 87;
    local cmd = string.format(
        "/usr/bin/vips copy '%s' '%s[Q=%d]'",
        src,
        cache_path,
        quality
    )

    os.execute(cmd)
    return ngx.exec(return_path)
end
