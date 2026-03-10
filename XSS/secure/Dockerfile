FROM php:8.1-apache

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    libzip-dev \
    && docker-php-ext-install zip \
    && rm -rf /var/lib/apt/lists/*

# Install MySQL extension and additional security tools
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Install Composer for dependency management
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Enable Apache modules
RUN a2enmod rewrite
RUN a2enmod headers

# Set working directory
WORKDIR /var/www/html

# Copy composer files first for better layer caching
COPY composer.json ./

# Install dependencies
RUN composer install --no-dev --optimize-autoloader

# Copy application files
COPY . /var/www/html/

# Re-run composer install to ensure vendor is populated after full copy
RUN composer install --no-dev --optimize-autoloader

# Set permissions
RUN chown -R www-data:www-data /var/www/html/
RUN chmod -R 755 /var/www/html/

# Expose port 80
EXPOSE 80
