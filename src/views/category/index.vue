<template>
  <div class="total">
    <div class="table-search">
      <p>搜索文章：</p>
      <el-input
        v-model="searchKeyword"
        placeholder="请输入文章名称"
        clearable
        style="width: 350px"
        @keyup.enter="handleSearch()"
      />
      <el-select v-model="searchKeyCategory" placeholder="请选择分类" clearable style="width: 180px">
        <el-option v-for="(category, index) in uniqueCategories" :key="index" :label="category" :value="category"></el-option>
      </el-select>
      <el-button type="primary" @click="handleSearch" :icon="Search">
        <el-icon style="margin-right: 5px"><Search /></el-icon>
        搜索
      </el-button>
      <el-button color="#3375b9" :dark="isDark" v-loading.fullscreen.lock="fullscreenLoading" @click="openFullScreen">
        <el-icon style="margin-right: 5px"><Loading /></el-icon>
        刷新
      </el-button>
      <el-select v-model="categoryValue" placeholder="请选择周刊期数" style="width: 180px; margin-left: 20px">
        <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" :disabled="item.disabled" />
      </el-select>
    </div>

    <div class="table-box">
      <div class="table-head">
        <el-button type="danger" @click="confirmDeleteCategories">
          <el-icon style="margin-right: 5px"><DeleteFilled /></el-icon>
          批量删除文章
        </el-button>
        <el-button type="primary" @click="drawer = true">
          <el-icon style="margin-right: 5px"><Edit /></el-icon>
          增添文章
        </el-button>
      </div>
      <el-table :data="originData" border stripe v-if="originData.length" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="40"></el-table-column>
        <el-table-column label="期数" width="90">
          <template #default="{ row }">
            {{ row.period }}
          </template>
        </el-table-column>
        <el-table-column label="分类" width="110">
          <template #default="{ row }">
            {{ row.category }}
          </template>
        </el-table-column>
        <el-table-column prop="name" label="名字" width="150" show-overflow-tooltip></el-table-column>
        <el-table-column property="description" label="概述" width="380" show-overflow-tooltip />
        <el-table-column prop="image" label="图片" width="180">
          <template #default="{ row }">
            <span v-if="row.image">
              <el-empty image="public/R-C.jpg" description="" style="width: auto; height: 80px; margin: auto" />
            </span>
            <span v-else> <el-empty :image-size="30" style="width: auto; height: 80px" /></span>
          </template>
        </el-table-column>
        <el-table-column prop="link" label="链接" width="250" show-overflow-tooltip>
          <template #default="scope">
            <a :href="scope.row.link" target="_blank" style="color: blue">{{ scope.row.link }}</a>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template #default="{ row }">
            <el-button type="primary" link @click="openDrawerWithData(row)">
              <el-icon style="margin-right: 5px"><View /></el-icon>
              查看
            </el-button>
            <el-button type="primary" link @click="changeDrawerWithData(row)">
              <el-icon style="margin-right: 5px"><EditPen /></el-icon>
              编辑
            </el-button>
            <el-button type="primary" link @click="confirmDelete(row.link)">
              <el-icon style="margin-right: 5px"><Delete /></el-icon>
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div v-else>
        <el-empty :image-size="347" />
      </div>
      <div class="demo-pagination-block">
        <el-pagination
          v-model:current-page="pageNum"
          v-model:page-size="pageSize"
          :page-sizes="[5, 10, 15, 20]"
          :small="small"
          :disabled="disabled"
          :background="background"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalItems"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </div>
  </div>

  <!-- 新增文章 -->
  <el-drawer v-model="drawer" title="新增文章" :with-header="false" custom-class="centered-drawer">
    <div class="demo-drawer__content centered-content">
      <el-card class="box-card">
        <h2 class="centered-title">新增文章</h2>
        <el-form ref="articleForm" :model="newArticle" label-width="80px" label-position="top">
          <el-col :span="24">
            <el-form-item label="期数" prop="period" :rules="rules.period">
              <el-select v-model="newArticle.period" placeholder="请选择对应期数">
                <el-option v-for="(period, index) in uniquePeriods" :key="index" :label="period" :value="period"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item label="类型" prop="category" :rules="rules.category">
                <el-select v-model="newArticle.category" placeholder="请选择类别">
                  <el-option
                    v-for="(category, index) in uniqueCategories"
                    :key="index"
                    :label="category"
                    :value="category"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="名字" prop="name" :rules="rules.name">
                <el-input v-model="newArticle.name" placeholder="请输入名字"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="概述" prop="description" :rules="rules.description">
                <el-input type="textarea" v-model="newArticle.description" placeholder="请输入概述"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="图片" prop="image" :rules="rules.image">
                <el-upload
                  action="your-upload-endpoint"
                  list-type="picture-card"
                  :on-success="handleUploadSuccess"
                  :on-remove="handleUploadRemove"
                  :file-list="fileList"
                  :limit="1"
                  accept=".jpg,.png"
                >
                  <template #default="{ file }">
                    <!-- 如果没有图片被选择，显示默认图片样式 -->
                    <el-image v-if="!file" src="public/image.png" class="image-slot" />
                    <img v-else :src="file.url" class="image-slot" />
                  </template>
                  <template #tip>
                    <div class="el-upload__tip">建议上传（500kb内） jpg/png 文件，或输入图片链接</div>
                  </template>
                </el-upload>
                <el-input v-model="newArticle.image" placeholder="输入图片 URL" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="链接" prop="link" :rules="rules.link">
                <el-input v-model="newArticle.link" placeholder="请输入链接"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24" class="text-right">
              <el-button type="primary" @click="confirmAdd">提交</el-button>
              <el-button @click="closeAdd">取消</el-button>
            </el-col>
          </el-row>
        </el-form>
      </el-card>
    </div>
  </el-drawer>

  <!-- 修改文章的抽屉 -->
  <el-drawer v-model="drawerChange" title="修改文章" :with-header="false">
    <div class="demo-drawer__content">
      <el-card class="box-card">
        <h2>修改文章</h2>
        <el-form ref="articleForm" :model="changeArticles" label-width="80px" label-position="top">
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item label="类型" prop="category" :rules="rules.category">
                <el-select v-model="changeArticles.category" placeholder="请选择类别">
                  <el-option
                    v-for="(category, index) in uniqueCategories"
                    :key="index"
                    :label="category"
                    :value="category"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="名字" prop="name" :rules="rules.name">
                <el-input v-model="changeArticles.name" placeholder="请输入名字"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="概述" prop="description" :rules="rules.description">
                <el-input type="textarea" v-model="changeArticles.description" placeholder="请输入概述"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="图片" prop="image" :rules="rules.image">
                <el-upload
                  action="your-upload-endpoint"
                  list-type="picture-card"
                  :on-success="handleUploadSuccess"
                  :on-remove="handleUploadRemove"
                  :file-list="fileList"
                  :limit="1"
                  accept=".jpg,.png"
                >
                  <template #default="{ file }">
                    <!-- 如果没有图片被选择，显示默认图片样式 -->
                    <el-image v-if="!file" src="public/image.png" class="image-slot" />
                    <img v-else :src="file.url" class="image-slot" />
                  </template>
                  <template #tip>
                    <div class="el-upload__tip">只能上传 jpg/png 文件，且不超过 500kb</div>
                  </template>
                </el-upload>
                <el-input v-model="changeArticles.image" placeholder="输入图片 URL" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <el-form-item label="链接" prop="link" :rules="rules.link">
                <el-input v-model="changeArticles.link" placeholder="请输入链接"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24" class="text-right">
              <el-button type="primary" @click="confirmChange">提交</el-button>
              <el-button @click="drawerChange = false">取消</el-button>
            </el-col>
          </el-row>
        </el-form>
      </el-card>
    </div>
  </el-drawer>

  <!-- 查看文章抽屉 -->
  <el-drawer v-model="OpenDrawer" title="查看文章" :with-header="false">
    <el-card>
      <div style="text-align: center">
        <template v-if="selectedRowData.image">
          <img :src="selectedRowData.image" style="max-width: 400px; max-height: 400px" @error="handleImageError" />
          <a :href="selectedRowData.image" target="_blank">
            {{ selectedRowData.image }}
          </a>
        </template>
        <template v-else>
          <div style="padding: 50px">暂无图片</div>
        </template>
      </div>
      <div class="info-container">
        <p><b>分类:</b> {{ selectedRowData.category }}</p>
        <p><b>名字:</b> {{ selectedRowData.name }}</p>
        <p><b>概述:</b> {{ selectedRowData.description }}</p>
        <p>
          <b>链接:</b>
          <a :href="selectedRowData.link" target="_blank">
            {{ selectedRowData.link }}
          </a>
        </p>
      </div>
      <div class="demo-drawer__footer" style="text-align: center">
        <el-button @click="OpenDrawer = false">关闭</el-button>
      </div>
    </el-card>
  </el-drawer>
</template>

<script setup>
import { ref, onMounted, computed, watch } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";

const drawer = ref(false);
const drawerChange = ref(false);
const searchKeyword = ref("");
const searchKeyCategory = ref("");
const selectedArticles = ref([]);
const isSearch = ref(false);

const data = ref(null);
// 获取文章数据
const fetchData = async path => {
  try {
    const response = await fetch(path);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching mock data:", error);
    throw error;
  }
};

// 定义全部json文件
const categoryValue = ref("Total");
const options = [
  { value: "Total", label: "全部文章" },
  { value: "第001期", label: "第001期" },
  { value: "第002期", label: "第002期" },
  { value: "第003期", label: "第003期" },
  { value: "第004期", label: "第004期" },
  { value: "第005期", label: "第005期" },
  { value: "第006期", label: "第006期" },
  { value: "第007期", label: "第007期" },
  { value: "第008期", label: "第008期" },
  { value: "第009期", label: "第009期" },
  { value: "第010期", label: "第010期" }
];

// 获取文章的路径计算
const getPath = categoryValue => {
  switch (categoryValue) {
    case "第001期":
      return "src/assets/json/08-16~08-20.老胡的周刊（第001期）.json";
    case "第002期":
      return "src/assets/json/08-23~08-27.老胡的周刊（第002期）.json";
    case "第003期":
      return "src/assets/json/08-30~09-03.老胡的周刊（第003期）.json";
    case "第004期":
      return "src/assets/json/09-06~09-10.老胡的周刊（第004期）.json";
    case "第005期":
      return "src/assets/json/09-12~09-17.老胡的周刊（第005期）.json";
    case "第006期":
      return "src/assets/json/09-19~09-24.老胡的周刊（第006期）.json";
    case "第007期":
      return "src/assets/json/09-27~10-03.老胡的周刊（第007期）.json";
    case "第008期":
      return "src/assets/json/10-04~10-10.老胡的周刊（第008期）.json";
    case "第009期":
      return "src/assets/json/10-11~10-17.老胡的周刊（第009期）.json";
    case "第010期":
      return "src/assets/json/10-18~10-24.老胡的周刊（第010期）.json";
    default:
      return null;
  }
};

const fetchAndStoreAllData = async () => {
  const categories = [
    "第001期",
    "第002期",
    "第003期",
    "第004期",
    "第005期",
    "第006期",
    "第007期",
    "第008期",
    "第009期",
    "第010期"
  ];
  const dataList = {};
  for (const category of categories) {
    const path = getPath(category);
    if (path) {
      try {
        const result = await fetchData(path);
        dataList[category] = result;
      } catch (error) {
        console.error(`Error fetching data for ${category}:`, error);
      }
    }
  }
  localStorage.setItem("dataList", JSON.stringify(dataList));
};

onMounted(async () => {
  // localStorage.removeItem("dataList");
  const storedData = localStorage.getItem("dataList");
  if (!storedData) {
    await fetchAndStoreAllData();
  }
  const allData = JSON.parse(localStorage.getItem("dataList"));
  console.log(allData);
  if (categoryValue.value === "Total") {
    let combinedData = {};
    // 遍历所有期数
    for (const period in allData) {
      // 遍历每个期数下的所有分类
      for (const category in allData[period]) {
        if (!combinedData[category]) {
          combinedData[category] = [];
        }
        // 为每条数据添加期数信息
        const categorizedData = allData[period][category].map(item => ({
          ...item,
          period // 添加期数信息
        }));
        // 将数据合并到对应的分类数组中
        combinedData[category] = combinedData[category].concat(categorizedData);
      }
    }
    data.value = combinedData;
  } else {
    const periodData = allData[categoryValue.value] || {};
    let categorizedData = {};
    for (const category in periodData) {
      categorizedData[category] = periodData[category].map(item => ({
        ...item,
        period: categoryValue.value // 添加期数信息
      }));
    }
    data.value = categorizedData;
  }
});

// 修改后的 watch 函数
watch(categoryValue, newValue => {
  const allData = JSON.parse(localStorage.getItem("dataList"));
  if (newValue === "Total") {
    let combinedData = {};
    // 遍历所有期数
    for (const period in allData) {
      // 遍历每个期数下的所有分类
      for (const category in allData[period]) {
        if (!combinedData[category]) {
          combinedData[category] = [];
        }
        // 为每条数据添加期数信息
        const categorizedData = allData[period][category].map(item => ({
          ...item,
          period // 添加期数信息
        }));
        // 将数据合并到对应的分类数组中
        combinedData[category] = combinedData[category].concat(categorizedData);
      }
    }
    data.value = combinedData;
    console.log(data.value);
  } else {
    const periodData = allData[newValue] || {};
    let categorizedData = {};
    for (const category in periodData) {
      categorizedData[category] = periodData[category].map(item => ({
        ...item,
        period: newValue // 添加期数信息
      }));
    }
    data.value = categorizedData;
    console.log(data.value);
  }
});

// 对获取到的数据做处理，比如将分类逐条添加到数据中去
const getAllItems = data => {
  let allItems = [];
  for (const category in data) {
    // 确保 data[category] 是数组
    if (Array.isArray(data[category])) {
      const categoryItems = data[category].map(item => ({ ...item, category }));
      allItems = allItems.concat(categoryItems);
    } else {
      console.error(`${category} 数据不是一个数组。`);
    }
  }
  totalItems.value = allItems.length;
  return allItems;
};

// 查看文章数据
const OpenDrawer = ref(false);
const selectedRowData = ref({
  category: "",
  name: "",
  description: "",
  image: "",
  link: ""
});
const openDrawerWithData = rowData => {
  selectedRowData.value = { ...rowData };
  OpenDrawer.value = true;
};

// 上传图片
const fileList = ref([]);
const handleUploadSuccess = (response, file, fileList) => {
  newArticle.value.image = response.url; // 假设返回的响应中有图片的 URL
  fileList.value = fileList;
};
const handleUploadRemove = (file, fileList) => {
  newArticle.value.image = "";
  fileList.value = fileList;
};
// 图片加载不出来就显示默认图片
const handleImageError = event => {
  event.target.src = "public/R-C.jpg";
};
// 取消新增文章的操作
const closeAdd = () => {
  drawer.value = false;
  resetForm();
  newArticle.value = {
    // 重置新增文章的数据
    category: "",
    name: "",
    description: "",
    image: "",
    link: ""
  };
  ElMessage({
    type: "info",
    message: "取消新增文章"
  });
};
const articleForm = ref(null);
const resetForm = () => {
  if (articleForm.value) {
    articleForm.value.resetFields();
  }
};
// 定义新增文章的input的提示语
const rules = ref({
  period: [
    { required: true, message: "请输入期数", trigger: "blur" },
    {
      validator: (rule, value, callback) => {
        // 检查是否为正整数
        if (!Number.isInteger(Number(value)) || Number(value) <= 0) {
          callback(new Error("期数必须是正整数"));
        } else {
          callback();
        }
      },
      trigger: "blur"
    }
  ],
  category: [{ required: true, message: "请选择类型", trigger: "blur" }],
  name: [{ required: true, message: "请输入名字", trigger: "blur" }],
  description: [{ required: true, message: "请输入概述", trigger: "blur" }],
  image: [
    { required: true, message: "请输入图片URL", trigger: "blur" },
    {
      validator: (rule, value, callback) => {
        if (!/^https:\/\/.*/.test(value)) {
          callback(new Error("请输入有效网址"));
        } else {
          callback();
        }
      },
      trigger: "blur"
    }
  ],
  link: [
    { required: true, message: "请输入链接", trigger: "blur" },
    {
      validator: (rule, value, callback) => {
        if (!/^https:\/\/.*/.test(value)) {
          callback(new Error("请输入有效网址"));
        } else {
          callback();
        }
      },
      trigger: "blur"
    }
  ]
});

// 新增文章数据
const newArticle = ref({
  period: "",
  category: "",
  name: "",
  description: "",
  image: "",
  link: ""
});
const confirmAdd = () => {
  ElMessageBox.confirm(`确认添加此文章吗？`, "警告", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  })
    .then(() => {
      addArticle();
    })
    .catch(() => {
      ElMessage({
        type: "info",
        message: "添加文章操作已取消"
      });
    });
};
const addArticle = () => {
  const period = newArticle.value.period; // 获取选择的期数
  const category = newArticle.value.category;
  const article = {
    name: newArticle.value.name,
    description: newArticle.value.description,
    image: newArticle.value.image,
    link: newArticle.value.link,
    period: period // 将期数信息添加到文章中
  };
  if (!data.value[category]) {
    data.value[category] = [];
  }
  data.value[category].push(article);
  updateLocalStorage();
  ElMessage({
    type: "success",
    message: "新增文章成功"
  });
  drawer.value = false;
};

// 更改文章数据
const changeArticles = ref({});
const originalLink = ref(""); // 保存原始 link
// 编辑文章数据
const changeDrawerWithData = rowData => {
  changeArticles.value = { ...rowData };
  originalLink.value = rowData.link; // 保存原始 link
  drawerChange.value = true;
};
const confirmChange = event => {
  event.stopPropagation();
  ElMessageBox.confirm(`确认更改此文章吗？`, "警告", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  })
    .then(() => {
      changeArticle();
    })
    .catch(() => {
      ElMessage({
        type: "info",
        message: "更改操作已取消"
      });
    });
};
const changeArticle = () => {
  const updatedArticle = { ...changeArticles.value };
  const allData = { ...data.value };
  // 更新文章数据
  for (const category in allData) {
    const index = allData[category].findIndex(item => item.link === originalLink.value);
    if (index !== -1) {
      const isCategoryChanged = allData[category][index].category !== updatedArticle.category;
      if (isCategoryChanged) {
        // 从原类别中移除文章
        allData[category].splice(index, 1);
        // 添加文章到新类别
        allData[updatedArticle.category] = allData[updatedArticle.category] || [];
        allData[updatedArticle.category].push(updatedArticle);
      } else {
        // 直接更新文章数据
        allData[category][index] = updatedArticle;
      }
      break; // 找到并处理文章后立即跳出循环
    }
  }
  data.value = allData;
  updateLocalStorage();
  ElMessage({
    type: "success",
    message: "文章更改成功"
  });
  drawerChange.value = false;
};

// 分页数据
const totalItems = ref(30);
const pageNum = ref(1);
const pageSize = ref(5);
const paginatedData = computed(() => {
  const start = (pageNum.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  const allItems = getAllItems(data.value);
  return allItems.slice(start, end);
});
const handleCurrentChange = async currentPage => {
  pageNum.value = currentPage;
};
//const handlePageSizeChange = newPageSize => {
//  pageSize.value = newPageSize;
//};

// 对四个主要分类做单独取出操作, Set 是一个集合，它与数组类似，但 Set 中的元素是唯一的，不能重复
const uniqueCategories = computed(() => {
  const categoriesSet = new Set();
  const allData = data.value || {};
  for (const category in allData) {
    categoriesSet.add(category);
  }
  return Array.from(categoriesSet);
});
// 计算属性，生成1到10期的期数
const uniquePeriods = computed(() => {
  const periods = [];
  for (let i = 1; i <= 10; i++) {
    periods.push(`第${i.toString().padStart(3, "0")}期`);
  }
  return periods;
});
// 对搜索功能的数据组进行计算
const filteredData = computed(() => {
  const allItems = getAllItems(data.value);
  const keyword = new RegExp(searchKeyword.value, "i");
  return allItems.filter(item => {
    const nameMatch = keyword.test(item.name);
    const categoryMatch = searchKeyCategory.value === "" || item.category === searchKeyCategory.value;
    return nameMatch && categoryMatch;
  });
});
// 定义表格数据渲染是哪个数据集
const originData = computed(() => {
  return isSearch.value ? filteredData.value : paginatedData.value;
});

const handleSearch = () => {
  isSearch.value = true;
  totalItems.value = filteredData.value.length;
};

const updateLocalStorage = () => {
  let allData = JSON.parse(localStorage.getItem("dataList"));
  if (categoryValue.value === "Total") {
    // 创建一个新的 allData 结构
    let newAllData = {};
    // 遍历 data.value 中的每个分类
    for (const category in data.value) {
      data.value[category].forEach(item => {
        const { period, ...rest } = item; // 提取 period 和剩余的字段
        // 初始化对应期数的对象，如果尚不存在
        if (!newAllData[period]) {
          newAllData[period] = {};
        }
        // 初始化对应期数下的分类数组，如果尚不存在
        if (!newAllData[period][category]) {
          newAllData[period][category] = [];
        }
        // 将数据添加到相应期数和分类中
        newAllData[period][category].push(rest);
      });
    }
    allData = newAllData;
  } else {
    // 确保在单个分类情况下添加 period 信息
    let newCategoryData = {};
    for (const category in data.value) {
      newCategoryData[category] = data.value[category].map(item => ({
        ...item,
        period: categoryValue.value // 添加期数信息
      }));
    }
    allData[categoryValue.value] = newCategoryData;
  }
  localStorage.setItem("dataList", JSON.stringify(allData));
};

// 删除功能操作
const confirmDelete = link => {
  ElMessageBox.confirm(`确认删除该文章吗？`, "警告", {
    confirmButtonText: "删除",
    cancelButtonText: "取消",
    type: "warning"
  })
    .then(() => {
      deleteAccount(link);
    })
    .catch(() => {
      ElMessage({
        type: "info",
        message: "删除操作已取消"
      });
    });
};
const deleteAccount = link => {
  const allData = data.value || {};
  for (const category in allData) {
    allData[category] = allData[category].filter(item => item.link !== link);
  }
  data.value = allData;
  updateLocalStorage();
  ElMessage({
    type: "success",
    message: "删除成功"
  });
};

// 多选删除操作
const handleSelectionChange = selection => {
  selectedArticles.value = selection.map(item => item.link);
};
const confirmDeleteCategories = () => {
  if (selectedArticles.value.length === 0) {
    ElMessage({
      type: "warning",
      message: "请选择要删除的文章"
    });
    return;
  }
  ElMessageBox.confirm(`确认删除选中的文章吗？`, "警告", {
    confirmButtonText: "删除",
    cancelButtonText: "取消",
    type: "warning"
  })
    .then(() => {
      deleteSelectedArticles();
    })
    .catch(() => {
      ElMessage({
        type: "info",
        message: "删除操作已取消"
      });
    });
};
const deleteSelectedArticles = () => {
  const allData = data.value || {};
  for (const category in allData) {
    allData[category] = allData[category].filter(item => !selectedArticles.value.includes(item.link));
  }
  data.value = allData;
  updateLocalStorage();
  ElMessage({
    type: "success",
    message: "批量删除成功"
  });
};

// 全屏刷新页面
const fullscreenLoading = ref(false);
const openFullScreen = () => {
  fullscreenLoading.value = true;
  setTimeout(() => {
    fullscreenLoading.value = false;
    searchKeyword.value = "";
    searchKeyCategory.value = "";
    isSearch.value = false;
    originData.value = paginatedData.value;
    const allItems = getAllItems(data.value);
    totalItems.value = allItems.length;
  }, 500);
};
</script>

<style scoped>
.total {
  position: relative;
}
.table-search {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
  padding: 5px 0;
  background-color: #ffffff;
}
.table-search p {
  margin-left: 20px;
  font-size: 15px;
  color: grey;
}
.table-search > .el-input {
  width: 300px;
  height: 90%;
  margin-right: 20px;
  margin-left: 20px;
}
.table-search > .el-button {
  width: 100px;
}
.table-search > .el-select {
  width: 100px;
  height: 90%;
  margin-right: 20px;
}
.table-box {
  height: 610px;
  margin: 10px 0 0;
  overflow: auto;
  background-color: #ffffff;
}
.table-box > .el-table {
  width: 97%;
  margin: 0 auto;
}
.table-box .table-head {
  padding: 15px 0;
  margin-right: 15px;
  background-color: #ffffff;
}
.table-box .table-head > .el-button {
  float: right;
  margin-right: 10px;
}
.demo-pagination-block {
  /* stylelint-disable-next-line shorthand-property-no-redundant-values */
  margin: -8px 30px 10px 0;
}
.info-container {
  margin: 20px 15px;
}
.info-container p {
  margin: 20px 3px;
}
.info-container b {
  margin-right: 10px;
}
</style>
