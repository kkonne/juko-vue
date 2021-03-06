<template>
    <div id="category">
        <div class="category-main container">
            <h1>Iiide {{ selectedCategory?.category_name }}</h1>

            <div v-if="categoriesList?.length">
                <div v-for="(category, i) in categoriesList" :key="i">
                    <div @click="handleCategorySelect(category)">
                        {{ category.category_name }}
                    </div>
                </div>
            </div>

            <div v-if="productsList.length">
                <div v-for="(product, i) in productsList" :key="i">
                    <div
                        @click="
                            routerService.routerLinkTo(
                                `/artikal/${product.product_slug}`
                            )
                        "
                    >
                        {{ product.product_name }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { ref, watch } from 'vue';
import router from '@/router';
import httpService from '@/services/http-service';
import routerService from '@/services/router-service';

export default {
    name: 'Products category',

    setup() {
        let selectedCategory = ref(null);
        let categoriesList = ref([]);
        let productsList = ref([]);
        let routeRef = ref(router.currentRoute);

        watch(routeRef, (newPath): void => {
            if (!newPath.fullPath.includes('proizvodi')) {
                return;
            }

            onInit();
            categoriesList.value = [];
            productsList.value = [];
        });

        const onInit = (): void => {
            getCategoryData();
        };

        const getCategoryData = (): void => {
            const currentRouteParam = routeRef.value.params.categorySlug;

            let API_URL: string;
            let isTopLevel: boolean;

            if (currentRouteParam !== 'svi') {
                API_URL = `categories/${currentRouteParam}`;
                isTopLevel = false;
            } else {
                API_URL = 'categories';
                isTopLevel = true;
            }

            httpService
                .get(API_URL)
                .then((response) => {
                    selectedCategory.value =
                        (!isTopLevel && response.data) || null;

                    const categoriesListData = isTopLevel
                        ? response.data
                        : response.data.children_categories;

                    if (!categoriesListData.length && !isTopLevel) {
                        categoriesList.value = [];
                        getCategoryProducts();
                        return;
                    }

                    categoriesList.value = categoriesListData;
                })
                .catch((error) => {
                    console.log(error);
                });
        };

        const getCategoryProducts = (): void => {
            const currentRouteParam = routeRef.value.params.categorySlug;
            const API_URL = `categories/${currentRouteParam}/products`;

            httpService
                .get(API_URL)
                .then((response) => {
                    productsList.value = response.data.products;
                })
                .catch((error) => {
                    console.log(error);
                });
        };

        const handleCategorySelect = (category): void => {
            routerService.routerLinkTo(category.category_slug);
        };

        onInit();

        return {
            selectedCategory,
            categoriesList,
            productsList,
            routeRef,

            getCategoryData,
            getCategoryProducts,
            handleCategorySelect,

            routerService,
        };
    },
};
</script>

<style lang="scss">
.category-main {
    @apply my-10;
}
</style>
