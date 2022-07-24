<template>
  <n-notification-provider :placement="'bottom-right'">
    <n-upload accept=".unitypackage" :default-upload="false" :show-file-list="false" @change="handleUploadChange">
      <n-upload-dragger>
        <div style="margin-bottom: 12px">
          <n-icon size="48" :depth="3">
            <package-icon />
          </n-icon>
        </div>
        <n-text depth="3">
          {{ filename !== "" ? filename : "Click or drag a file to this area." }}
        </n-text>
      </n-upload-dragger>
    </n-upload>

    <tree :data="data" />
  </n-notification-provider>
</template>

<script setup lang="ts">
import { Archive } from "libarchive.js/main.js";
import workerUrl from "libarchive.js/dist/worker-bundle.js?url";
import { h, ref, Ref } from "vue";
import {
  NText,
  NIcon,
  NUploadDragger,
  NUpload,
  NNotificationProvider,
  TreeOption,
  UploadFileInfo,
} from "naive-ui";
import {
  Package as PackageIcon,
  Folder as FolderIcon,
  File as FileIcon,
  FileCertificate as LicenseIcon,
  CSharp as CSharpIcon,
  Box as PrefabIcon,
  Photo as ImageIcon,
  BallVolleyball as MaterialIcon,
  Markdown as MarkdownIcon,
  FileText as FileTextIcon,
  FileCode as FileCodeIcon,
  FileZip as FileZipIcon,
  Binary as BinaryIcon,
  PlayerPlay as PlayerPlayIcon,
  Sitemap as SitemapIcon,
} from "@vicons/tabler";
import { Unity as UnityIcon, Heart as HeartIcon } from "@vicons/fa";
import Tree from "./components/Tree.vue";

Archive.init({ workerUrl });

interface Assets {
  id?: string;
  path: string;
  isFile: boolean;
  preview?: string;
  name?: string;
  prefix?: any;
  children?: Assets[];
}

const data: Ref<TreeOption[]> = ref([]);
const filename = ref("");

const handleUploadChange = async (options: { file: UploadFileInfo }) => {
  if (!options.file) return;
  const file = options.file.file!;

  filename.value = file.name;

  const archive = await Archive.open(file);
  const arr: Assets[] = [];

  const objs = await archive.extractFiles();
  for (const id in objs) {
    const pathname = await ((objs[id] as any).pathname as File).text();
    const isFile = Object.prototype.hasOwnProperty.call(objs[id], "asset");
    const pn = pathname.split("/");
    const info: Assets = {
      id,
      name: pn[pn.length - 1],
      path: pathname,
      isFile,
    };
    if (Object.prototype.hasOwnProperty.call(objs[id], "preview.png"))
      info.preview = (await toBase64(
        (objs[id] as any)["preview.png"] as File
      )) as string;
    arr.push(info);
  }
  data.value = loadAssets(arr);
};

const loadAssets = (arr: Assets[]) => {
  let t = { name: "", path: "", isFile: false, children: [] };
  arr.forEach((v) => {
    ((now: Assets, info: Assets) => {
      const ps = info.path.split("/");
      for (let i = 0; i < ps.length; i++) {
        let nt: Assets;
        const fn = now.children?.find((v) => v.name === ps[i]);
        if (!fn) {
          nt = {
            name: ps[i],
            path: now.path + "/" + ps[i],
            isFile: false,
            children: [],
          };
          if (i === ps.length - 1 && info.isFile) {
            nt.isFile = true;
            nt.id = info.id;
            nt.preview = info.preview;
            delete nt.children;
            now.children?.push(nt);
          } else now.children = [nt].concat(now.children!);
          now.children?.sort((a, b) => (a.name! < b.name! ? -1 : 1));
          nt.prefix = () => h(NIcon, {}, { default: () => h(getIcon(nt)) });
          now = nt;
        } else now = fn;
      }
    })(t, v);
  });
  return t.children;
};

const toBase64 = (file: File) =>
  new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () =>
      resolve((reader.result as string).replace(file.type, "image/png"));
    reader.onerror = (error) => reject(error);
  });

const getIcon = (a: Assets) => {
  if (!a.isFile) return FolderIcon;
  const t = a.name!.split(".");
  if (t[0].toUpperCase() === "LICENSE") return LicenseIcon;
  switch (t[t.length - 1]) {
    case "png":
    case "psd":
    case "jpg":
      return ImageIcon;
    case "cs":
      return CSharpIcon;
    case "unity":
      return UnityIcon;
    case "prefab":
      return PrefabIcon;
    case "anim":
      return PlayerPlayIcon;
    case "controller":
      return SitemapIcon;
    case "md":
      return MarkdownIcon;
    case "dll":
      return BinaryIcon;
    case "mat":
      return MaterialIcon;
    case "hlsl":
    case "shader":
    case "json":
      return FileCodeIcon;
    case "txt":
      return FileTextIcon;
    case "zip":
    case "rar":
    case "tar":
    case "gz":
      return FileZipIcon;
    case "fbx":
      return HeartIcon;
    default:
      break;
  }
  return FileIcon;
};
</script>

<style>
#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
}
</style>